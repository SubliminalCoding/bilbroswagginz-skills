---
name: memguard
description: >-
  Prevent memory-overload freezes when STARTING memory-heavy apps on a unified-memory
  box (DGX Spark / GB10, Jetson). Use before launching/restarting any model, finetune,
  transcription job, or dev server; when "the box froze when I started X"; when wiring a
  new service's ExecStart or a cron line on such a host; or when asked to "wrap this so it
  can't freeze the machine" / "preflight this heavy job" / "is there room to start vLLM".
  Wrap launches as `memguard run -- CMD`. Pairs with system-performance-remediation,
  if installed (recovery, when the box is ALREADY thrashing).
metadata:
  type: devops
---

# memguard: the memory-overload guardian

> **Core insight:** On a unified-memory box the GPU and system RAM are ONE pool, and a
> GPU model load is charged through the driver where the cgroup can't see it (a verified
> 8GiB cudaMalloc moved `memory.current` by only 87MiB). So you can't *cap* your way to
> safety; you have to *not start* the launch that won't fit. The load-bearing defense is
> a preflight admission gate plus a **one-GPU-heavy-app-at-a-time mutex**; the cgroup cap
> and earlyoom are nets underneath, not the primary guard.

## The failure mode in three lines

1. GPU/unified allocations count against host RAM but are invisible to memcg and to
   `nvidia-smi` (reports `N/A` on UMA; there's no tegrastats on the GB10).
2. Start a heavy app on a near-full pool → the kernel can't reclaim the GPU pages →
   it drags the slow disk swapfile in → PSI `full` climbs → **livelock before any clean
   GPU-OOM ever fires.** The whole box freezes.
3. The only gauge that tells the truth is `/proc/meminfo` `MemAvailable` + `SwapFree`
   (and `/proc/pressure/memory` for early warning). Trust those, never `nvidia-smi`.

## The one rule

**Never launch a memory-heavy app bare on this box.** Wrap it:

```
memguard run --profile my-api -- ./api/start.sh
memguard run --peak 40 -- python train.py          # ad-hoc, no profile
memguard check --profile llm-server                 # dry-run: exit 0/10/20/30
memguard status                                     # mem, PSI, gate, live scopes
```

`memguard run` preflights, enforces the gpu-heavy mutex, then launches CMD inside a
rootless cgroup scope (`MemoryHigh`/`MemoryMax`, and `MemorySwapMax=0` by default; a
per-profile override is allowed, e.g. 8G for a host-swap-tolerant app) with a
raised oom_score so a runaway dies in its own scope, not box-wide. It blocks for CMD's
life (drop-in for a terminal, an `ExecStart=`, or a cron line). Exit codes: `check`
returns 0/10/20/30 (run/degrade/queue/abort); `run` returns the child's exit code on
admit, 20 on queue-expired, 30 on hard abort.

## Automatic enforcement (so nobody has to remember the prefix)

`memguard install-shell` adds a hook to `~/.bashrc` that auto-wraps **known-heavy**
commands you type (ledger apps + `ollama run/serve`, vLLM, finetune, `uvicorn`,
`pnpm dev`, `*/start.sh`, `python …train/finetune/serve`). Light commands, and anything
with a pipe/redirect/list, run untouched. Bypass one launch with `MG=0 <cmd>`. This
closes the opt-in hole; the watchdog + earlyoom still cover anything unwrapped.

## Integration per launch style

```
# systemd unit:   ExecStart=%h/bin/memguard run --profile X -- /path/to/app
# cron:           * * * * * %h/bin/memguard run --profile X -- /path/to/job
# shell / NTM:    memguard run --profile X -- <cmd>   (or just rely on install-shell)
# agent (gate):   memguard check --profile X --json   # branch on .decision before launching
```

Any agent runtime launching heavy work on the box MUST route through
`memguard run`, or gate with `memguard check` first. Record this as authoritative
in your shared cross-agent memory file.

## The footprint ledger (`~/.config/memguard/apps.conf`)

One row per app: `app|class|peak_gb|low_gb|mutex_group|swapmax|oom_adj|low_env|note`.
Add a profile for a new app before wrapping it. **Estimate `peak_gb` generously.** It's
the admission gate's only knob, and the GPU side is invisible, so under-guessing is the
one way to defeat the guard. Unprofiled launches fall to `[default]` (8G, conservative).
`class`: `protected` (counted in the math, never auto-killed) vs `kill-safe` (capped,
may be shed under RED) vs `warn` (capped, never auto-killed). `memguard profiles` lists it.

## The 3+1 layer model

| Layer | Mechanism | Reliable? |
|---|---|---|
| L1 preflight admission + **gpu-heavy mutex** | refuse/queue/degrade before launch on MemAvailable+SwapFree; only one of {vLLM, big-Ollama, finetune} GPU-resident | **primary: this is what prevents the freeze** |
| L2 per-app cgroup scope, `MemorySwapMax=0` (default; per-profile override) | runaway dies in its own scope, no box-wide swap thrash | catches HOST-side runaway only (Python heap, dataloaders, mmap); **GPU loads are invisible to it** |
| watchdog `memguard monitor` | PSI poller; at RED flips the admission-freeze gate (non-destructive, catches unwrapped pressure too) + best-effort sheds largest kill-safe scope | gate-flip reliable; kill best-effort (a userspace reaper can be scheduled out under the very livelock it guards) |
| L3 earlyoom | the one kernel-adjacent reaper; hard floor for anything launched unwrapped | last resort |

## Decision reference

- Thresholds (env-overridable): projected-free `≥ SOFT=16G` → RUN_NORMAL; `≥ HARD=8G` →
  DEGRADE; `MemAvailable < FLOOR=4G` → ABORT; `SwapFree < SWAP_MIN=4G` → never RUN_NORMAL.
- PSI watchdog: WARN at `some avg10>15` / `full>5`; RED at `full>40` sustained 3 ticks
  (or `MemAvailable<HARD AND SwapFree<SWAP_MIN`) → freeze admissions; RECOVER after 8 calm
  ticks (calm = `MemAvailable>SOFT AND SwapFree>SWAP_MIN AND full PSI ≤ WARN`, decoupled
  from the noisy `some` signal so the gate actually auto-thaws).
- DEGRADE applies the profile's `low_env` (e.g. `<APP>_FORCE_CPU=1`) and caps the
  scope at `low_gb`: smaller footprint instead of refusing outright. A profile with
  `low_gb=0` (e.g. finetune, which can't shrink) keeps its full cap rather than degrading.
- `memguard pause [reason]` / `resume`: manually freeze admissions before a known-huge
  job (e.g. a finetune) or while debugging.

## Composition + hard rules

- **If the box is ALREADY thrashing, memguard won't save you.** Run
  `/system-performance-remediation` (if installed; otherwise stop the heaviest
  process by hand) to recover, *then* relaunch under memguard.
- **Exactly one reaper.** earlyoom only. Do NOT add nohang or systemd-oomd alongside it
  (double-kills). systemd-oomd is a future swap-not-stack upgrade, not an addition.
- **gpu-heavy mutex:** stop one big model server before starting another (e.g. stop
  Ollama to free its memory before vLLM comes up). memguard enforces it; don't run
  two big models.
- Keep `vm.swappiness=10` and keep swap present; add modest zram (see `memguard setup`)
  to widen the watchdog/earlyoom reaction window.

## First-time setup

```
memguard setup     # installs the user watchdog (rootless); prints the root steps:
                   #   earlyoom retune (-m 8,4 -s 5,2 + avoid/prefer) and zram
memguard verify    # smoke-test the primitives
```

## Anti-patterns

- Launching a heavy app bare "just this once". That's the one that freezes the box.
- Trusting `nvidia-smi`/`cudaMemGetInfo` for headroom (N/A / underreports on UMA).
- Setting `peak_gb` tight to "fit more". The GPU side is uncounted; generous is safe.
- Relying on `MemoryMax` to stop a GPU OOM; it can't see the GPU allocation.
- Running two gpu-heavy models at once because "there's RAM free right now."
- Adding a second OOM daemon for "extra safety". It double-kills.

## References

- system-performance-remediation, if installed: the RECOVERY counterpart (use when
  already thrashing)
- `~/bin/memguard`: the CLI · `~/.config/memguard/apps.conf`: the ledger
