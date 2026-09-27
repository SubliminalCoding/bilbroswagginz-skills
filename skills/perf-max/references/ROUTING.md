# Routing: Perf Max

Which installed skill does what. Skills outside this pack (verify, run,
testing-*, system-performance-remediation, deadlock-finder-and-fixer, deps-update,
library-updater) are optional: use them if installed, otherwise do that step directly. perf-max adds the sequencing + proof + measurement;
these skills do the profiling and optimizing.

## Phase 1 PROFILE: find the real hotspots

| Need | Skill |
|---|---|
| Rank hot paths by CPU / memory / I/O / contention under real usage | hotspot-scout |

Output is a *scored* target list. Optimize the top of it, nothing else. No profiler
output → do NOT optimize (establish a workload and profile first).

## Phase 2 OPTIMIZE: fix the scored hotspots

| Need | Skill |
|---|---|
| Profile-driven optimization with behavior proofs (the actual optimizing) | optimize-proof |
| Grind the ranked list systematically, progressive deepening | deepen (wrapping optimize-proof) |

One hotspot at a time, each under the green ratchet.

## Phase 3 PROVE: faster AND unchanged AND actually faster

- **verify** / **run** (if installed; otherwise run the workload yourself): run the workload; confirm the speedup is real.
- **testing-golden-artifacts:** outputs byte-identical before/after (behavior proof).
- **testing-metamorphic:** relational/property invariants hold after the change.
- Record before/after numbers (p95, throughput, alloc, IOPS). No measurable win or any
  behavior change → revert the change. Fast-and-wrong is wrong; slower-with-no-payoff
  is just added complexity.

## Escalate: when the bottleneck isn't the code you're optimizing

| Symptom | Skill |
|---|---|
| The MACHINE is bogged (load, swap, IO pressure, disk, cache bloat) | system-performance-remediation |
| The cost is lock contention / a concurrency stall | deadlock-finder-and-fixer |
| A slow or outdated dependency is the hotspot | deps-update / library-updater |
| The hotspot is tangled/unreadable code; the problem is complexity, not latency | simplify-proof (optimize-proof's sibling: characterization test first, behavior locked, measured complexity drop or revert) |

## Staged prompts

```
Phase 0: re-read AGENTS.md/README and understand the architecture. establish a
         representative workload to profile against, and a green baseline. checkpoint.

Phase 1: run hotspot-scout against that workload. give me the SCORED
         hotspot list (CPU/mem/IO/contention). do not optimize anything yet.

Phase 2: optimize the top hotspot with optimize-proof, one at a time,
         under the green ratchet. use deepen to work down the list.

Phase 3: for each change, prove behavior is unchanged (golden/metamorphic + run it)
         and measure the before/after win. no measurable gain or any behavior change?
         revert it.

Phase 4: re-profile; the bottleneck has moved. keep going until two passes find no
         material win left.
```
