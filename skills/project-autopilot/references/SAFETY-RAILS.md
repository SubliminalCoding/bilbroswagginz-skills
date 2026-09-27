# Safety Rails

The rails that make "off to the races" safe. These override enthusiasm.

## Batch discipline

- **One skill, one batch, one tick.** Never run two mutating skills concurrently
  in solo mode. (Swarm mode delegates concurrency to ntm + agent-mail file
  reservations, if installed; the autopilot still dispatches one *kind* of work per pane.)
- **Batch size:** ≤ 5 beads or ≤ ~10 files touched per tick, whichever comes
  first. Mass-dispatch is the documented ideation failure mode; small batches
  with green checks between them is the antidote.
- **Late-stage guard:** if the working tree already has >10 modified files or
  >3 in-flight beads when autopilot is launched, do NOT start a new diagnosis
  pass. Finish/stabilize what's in flight first.

## Swarm-mode tending (when IMPLEMENT runs as an ntm swarm)

The controller (CC) tends the swarm; it does not just launch and walk away.

- **No external tool is a hard dependency.** Codex, ntm, a specific model sub:
  all are accelerators, never preconditions. Probe availability at IMPLEMENT time
  (references/EXECUTION.md "Execution backend: detect & degrade") and pick the highest live
  rung; solo-sequential CC is the always-available floor. The loop must NEVER
  stall, fail, or report blocked because a Codex sub lapsed or ntm is down. It
  degrades to a working engine and keeps going, noting the slower tradeoff.

- **Canonical shape:** ~4 cod + 2 cc instances via ntm + vibing-with-ntm (if installed); CC as
  top-level controller running its looper every ~4 min to pass fresh instructions
  to *idle* agents, assignments guided by `bv` triage of open beads. (Counts and
  cadence tune per project; a proven default is 4 cod + 2 cc, 4-min looper.)
  cod instances MUST be pinned to the newest model (`--cod=N:gpt-5.5` or newer):
  codex-cli silently defaults to an older model otherwise; verify the pane label.
- **Stalled-bead recovery (prevents permanent wedging):** every few looper ticks,
  scan for beads marked `in_progress` by agents that are long-dead with no recent
  work on them, and reset those to `open` so a live agent can pick them up. A
  dead agent's claim is not a lock; without this, work silently strands forever.
- **Build discipline:** use `rch` for all builds/tests in swarm mode if it's
  installed (otherwise your normal local build or CI). Avoid
  excessive build contention from multiple agents building the same project
  concurrently, and periodically clear stale build artifacts so the box doesn't
  run out of disk (if it does, use system-performance-remediation if installed, or clear caches by hand).
- **Review transition:** when beads drain, flip the swarm into vibing-with-ntm
  code-review mode (cc + cod) rather than tearing it down, then apply the
  saturation gate (CONVERGENCE.md).

## Memory guards (consult BEFORE acting, not after failing)

- `cm` playbooks + trauma guard (if installed; otherwise the project's own notes)
  before each EXECUTE step. Never repeat a documented mistake.
- The operator's standing rules apply inside the loop. These examples come from
  one real setup; keep the ones that match your setup, add your own:
  - If a local model host runs on unified-memory hardware (GB10-class): one
    model loaded at a time; `--gpu-memory-utilization ≤ 0.75`
  - No secrets in chat/logs/commits: env files + silent prompts only
  - Repos live at `~/<name>`, never /tmp
  - Skills are symlinked across runtimes, never copied

## Decision inbox protocol

File: `<project>/.autopilot/DECISIONS.md` (create on first ask-user item).

```
## [open] <one-line decision>
- context: <why this came up, with evidence>
- options: <2-3 concrete options, recommended one first>
- blocked work: <bead IDs waiting on this, if any>
- raised: <date> tick <N>
```

Rules:
- The loop NEVER answers its own inbox items, even obvious-seeming ones.
- Inbox items don't block unrelated work; route around them.
- If >50% of remaining gaps are blocked on one decision, surface that in the
  status line / report header: that decision is the bottleneck.

## Forbidden actions (regardless of routing-map mode)

- `git push --force`, history rewrites, branch deletion
- Publishing: releases, app-store submissions, production deploys, sending
  emails/messages/posts, charging cards
- Deleting user data, dropping tables, destructive migrations
- Rotating/creating credentials
- Editing files outside the target project (except its own `.autopilot/` state,
  beads, and its run log)
- Disabling, skipping, or weakening tests to get to green

## Escalation triggers (stop the loop and ping the operator, don't just inbox)

- Any secret/credential discovered hardcoded
- Evidence of data loss or corruption
- The test suite was already red at launch (the ratchet has no baseline; it needs
  a human call on what "green" means here)
- Dependency with a known critical CVE in a shipped product
