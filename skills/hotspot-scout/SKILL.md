---
name: hotspot-scout
description: >-
  Profile a system under its REAL workload and emit a SCORED, ranked hotspot list:
  hot paths by CPU, memory, I/O, and lock/contention, each with measured evidence
  (flamegraph / sampler / timing), p95/p99 where latency matters, and a headroom
  estimate. Measures and ranks; never optimizes. Use when "profile this", "why is
  it slow", "find the bottleneck", "where's the time going", "rank the hot paths",
  "flamegraph", "p95 is bad", or when perf-max / project-autopilot need their
  profiling stage.
---

# Hotspot Scout

Point it at a system and a real workload and it comes back with the one artifact
performance work actually needs: a ranked target list. Not a wall of profiler
output, but a scored table where every row is a hot path, its measured cost on the
real workload, the evidence behind the number, and an honest estimate of how much
of that cost is removable. The deliverable is *where to aim*. Pulling the trigger
is someone else's job (optimize-proof, or optimize-proof under
perf-max).

> **Core Insight:** Collecting samples is the easy 10%; every stack has a
> one-command profiler. The judgment is converting samples into a *ranked bet*:
> what does each hot path cost on the REAL workload (absolute, not just %), and
> how much of that cost is actually removable? Score = **cost × removable
> fraction**, with Amdahl as the ceiling. A path eating 40% of the run can never
> buy more than a 1.67× speedup, and a 90%-of-3ms path isn't a target at all.
> And profile the *waits*, not just the work: an app that's slow at 8% CPU isn't
> computing, it's blocked on I/O, allocation, or a lock. CPU-only profiling
> there ranks the wrong list.

## Hard boundary

This skill MEASURES and RANKS. It does not change code, flags, configs, or
indexes, not even the "obvious one-liner." The moment it edits, the next
profile measures its own contamination. Emit the list, stop.

## Preconditions

1. **A real workload exists.** A command, script, replayed traffic, or live
   process that represents actual usage. No workload → establish one FIRST (with
   the operator if it needs judgment); profiling a microbenchmark ranks a fiction.
2. **A production-shaped build.** Rust: `--release` with debug symbols
   (`[profile.release] debug = true`). Node: no inspector overhead in the timed
   runs. Python: same interpreter/venv as prod. Profiling a debug build ranks
   the compiler, not the code.
3. **A quiet box.** Kill competing heavy processes before baselining. On a
   unified-memory box (e.g. an NVIDIA DGX Spark/GB10), heavy profiled runs go
   through memguard when it is installed (`memguard run --profile <X> -- CMD`),
   never bare, and memory truth comes from `/proc/meminfo` / PSI, never nvidia-smi (N/A on
   unified memory). On other machines, run directly with the platform's normal
   profiler.

## The Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  ORIENT    Read intent + the code's shape (entry points, hot loops, I/O
                   surfaces, concurrency model). Pin down the exact workload
                   invocation and the metric that matters (wall, p95, throughput).
Phase 1  BASELINE  Run the workload ≥3 times, warm-up excluded. Record wall time,
                   p50/p95/p99 (latency paths), throughput, RSS peak. This is the
                   denominator every score divides by. High run-to-run variance
                   (>10%) → quiet the box or raise N before trusting anything.
Phase 2  CPU       On-CPU sampling profiler → flamegraph (per-stack tool routing
                   in references/PROFILERS.md). Note self vs cumulative; wide
                   flat plateaus are the targets, deep narrow towers usually
                   aren't.
Phase 3  WAITS     The other three dimensions (required, not optional):
                   MEM   allocation profile, GC/alloc pressure, RSS growth, leaks.
                   I/O   syscall summary (strace -c / iostat), fsync counts,
                         sequential-vs-random, N+1 queries, network round-trips.
                   LOCK  off-CPU / blocked time, lock wait, pool exhaustion,
                         event-loop lag, async executor stalls.
                   Skip rule: a dimension may be skipped ONLY with evidence it's
                   cold (e.g. CPU-bound at ~100% with flat RSS and near-zero
                   iowait); record the evidence for the skip.
Phase 4  SCORE     For each candidate hot path:
                   cost      = share of baseline, AND absolute (ms / MB / calls)
                   ceiling   = Amdahl: max speedup if cost → 0 = 1/(1−share)
                   removable = honest fraction actually winnable (algorithmic
                               class change, caching, batching, dropping work)
                               with one line of WHY
                   score     = cost × removable → rank descending
                   confidence: high (seen in every run) / med / low (one run,
                   or profiler overhead may distort); low-confidence rows get
                   re-measured or flagged, never silently ranked.
Phase 5  EMIT      Write the artifact set (contract below). Sanity gate before
                   declaring done: do the top hotspots sum plausibly against the
                   baseline (no double-counting parent+child frames)? Does the
                   #1 row name a code location an optimizer can open?
```

## Contract (what perf-max / optimize-proof / project-autopilot rely on)

perf-max Phase 1 runs this skill against its workload and consumes the scored
list directly; project-autopilot routes "'slow' but unprofiled" here
automatically. The interface is fixed:

- `<project>/.hotspots/HOTSPOTS.md`: the scored, ranked table (human-readable):

  ```
  | RANK | HOTSPOT (file:symbol)       | DIM  | COST              | HEADROOM            | EVIDENCE             | CONF |
  |------|-----------------------------|------|-------------------|---------------------|----------------------|------|
  | 1    | db.py:fetch_rows (N+1)      | io   | 61% · 4.9s of 8.1s| ceil 2.6× · ~85% rm | evidence/strace.txt  | high |
  | 2    | parse.rs:tokenize           | cpu  | 22% · 1.8s        | ceil 1.3× · ~50% rm | evidence/cpu.svg     | high |
  ```

  Plus the baseline block (workload command, runs, wall, p50/p95/p99, RSS peak)
  and a per-row note: why this much is removable, and what p95/p99 it drives.

- `<project>/.hotspots/hotspots.json`: same rows machine-readable
  (`rank, hotspot, dimension, cost{share_pct, absolute}, headroom{ceiling_x,
  removable_pct}, evidence[], confidence, note` + `baseline{}`), so
  optimize-proof / perf-max can iterate the list without parsing prose.
- `<project>/.hotspots/evidence/`: flamegraphs, sampler dumps, syscall
  summaries, raw timing runs. Every COST cell traces to a file here.
- Exit state: list emitted, **nothing optimized**. The top row is where
  optimize-proof starts.

## THE EXACT PROMPT

```
/hotspot-scout
Target:     <project path or hot subsystem>
Workload:   <the real usage to run: command / script / traffic to replay>
Metric:     <what "slow" means here: wall | p95 latency | throughput | RSS>
Dimensions: <optional: cpu | mem | io | lock | all (default all)>
```

## Anti-Patterns

- Optimizing anything, even the "free" one-liner. The skill ends at the list.
- Profiling a microbenchmark and presenting it as the workload: tuning a fiction.
- One run, or warm-up/JIT included in the numbers: measuring noise.
- CPU-only profiling of an app that's mostly waiting. The slow part is off-CPU.
- Percentages without absolutes. "38% of 4ms" is not a target; cost cells carry both.
- Headroom = cost, claiming 100% removable. Something replaces the work; say what fraction wins.
- Ranking parent and child frames as separate hotspots: double-counted cost, inflated list.
- Profiling a debug/instrumented build and shipping those numbers.
- Bare heavy runs on a unified-memory box (skipping memguard when it's installed), or nvidia-smi for memory truth on unified memory.
- A "hotspot" with no evidence file behind it. If it isn't measured, it isn't on the list.

## References

- [PROFILERS.md](references/PROFILERS.md): per-stack tool routing (Node/V8,
  Python, Rust, browser, system-wide, DB, local-LLM) + unified-memory (DGX Spark/GB10)
  measurement notes
