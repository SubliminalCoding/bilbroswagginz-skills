---
name: perf-max
description: >-
  Maximize a project's performance, exhaustively, by orchestrating profiling +
  optimization skills: always optimizing a profiler-scored hotspot (never by vibe),
  proving each change keeps behavior identical under a green ratchet, measuring the
  actual speedup, and re-profiling until the wins dry up. Use when "make it fast",
  "optimize this", "it's slow", "reduce latency/p95", "improve throughput", or a
  late-stage performance pass before shipping.
---

# Perf Max

The "fast" pillar of the triad (its siblings: **bug-zero** = correct, **ux-maximizer**
= beautiful). Point it at working code and drive the latency down by *measurement*,
not guesswork. It conducts the profiling + optimization skills you already have and
adds the discipline they need: profile first, optimize the real hotspot, prove
behavior is unchanged, measure the win, and re-profile because the bottleneck moves.

> **Core Insight:** Performance work fails two ways: optimizing by vibe (you speed
> up code that wasn't the bottleneck) and optimizing too early (you lock in structure
> the project hasn't earned). This vertical refuses both: every change targets a
> *profiler-scored* hotspot, and the whole pass is gated to *settled* code. Like its
> siblings, it's a successive-prompter with a **green ratchet**: a "faster" change
> that alters behavior or reds a test is reverted, not kept.

## What perf-max adds (the only new logic)

1. **Profile-first discipline:** no optimization without a measured target. Vibe-
   tuning is forbidden; the profiler picks the work.
2. **Behavior proof per change:** output identical + tests green, or the "speedup"
   reverts. Fast and wrong is just wrong.
3. **Measure the win:** before/after numbers per change; an optimization with no
   measurable gain gets reverted (don't keep complexity that didn't pay).
4. **Re-profile loop:** the hotspot moves after you fix the top one; re-profile and
   stop on saturation (diminishing speedup per effort), not guesswork.

## Preconditions / guards

1. **Late-stage only.** Optimization is a *maturity activity*. If features are still
   churning (>10 modified files OR >3 in-flight beads, or core goals not yet WORKING),
   STOP. Premature optimization wastes passes on code about to be rewritten and can
   lock in the wrong structure. Profile a thing you're about to ship.
2. **A representative workload exists.** You can't optimize what you can't measure.
   If there's no realistic benchmark/usage to profile against, establish one FIRST;
   that's the precondition, not optional.
3. **Green baseline + checkpoint** before any change.

## The Pipeline (staged: issue each step only when the prior finishes)

```
Phase 0  ORIENT    Read code + intent; establish a representative workload + a green
                   baseline; checkpoint. (Re-read AGENTS.md/README, deep-investigate.)
Phase 1  PROFILE   hotspot-scout → a SCORED hotspot list (CPU / mem
                   / I/O / contention) under real-world usage. No targets → no work.
Phase 2  OPTIMIZE  optimize-proof on the top hotspot(s), ONE at a time,
                   under the green ratchet. deepen to grind the list
                   systematically with progressive deepening.
Phase 3  PROVE     Behavior proof (output identical, tests green; golden/metamorphic
                   tests where apt) AND measure the win (before/after p95/throughput).
                   No measurable gain or any behavior change → revert that change.
Phase 4  RE-PROFILE Re-profile: the bottleneck has moved. Stop when K=2 passes find
                   no material win left (saturation). Else → Phase 2 on the new top.
```

## Skills it orchestrates

100% your installed arsenal: perf-max adds sequencing + proof, not profilers. Full
map + staged prompts in [references/ROUTING.md](references/ROUTING.md). In brief:
**profile** with hotspot-scout; **optimize** with
optimize-proof (driven by deepen); **prove** with optimize-proof's behavior proofs,
plus verify / testing-golden-artifacts / testing-metamorphic if installed (behavior
unchanged); **escalate** to system-performance-remediation (the *machine* is the
bottleneck, not the code), deadlock-finder-and-fixer (lock contention), deps-update
(a slow dep), each if installed (otherwise handle it directly), or
simplify-proof (the profile keeps pointing at tangled code: the real cost is
complexity, not compute; it's optimize-proof's sibling, same proof spine, objective
inverted to a measured complexity drop).

## Shared mechanisms (inherited from project-autopilot)

Green ratchet, decision inbox, saturation convergence, capability-aware execution,
as in project-autopilot; see its CONVERGENCE.md / SAFETY-RAILS.md / EXECUTION.md
rather than duplicating. **Integration:** project-autopilot's performance lens /
late-stage optimization pass routes here for the exhaustive treatment, exactly as the
UX lens routes to ux-maximizer and the correctness lens to bug-zero.

## THE EXACT PROMPT

```
/perf-max
Target: <project path or hot subsystem>
Workload: <the benchmark / realistic usage to profile against>
Mode: solo | swarm
```

## Anti-Patterns

- Optimizing by vibe instead of a profiler-scored target: the cardinal sin.
- Running it early, on a still-churning codebase (premature optimization).
- Keeping a "faster" change that altered behavior or reds a test.
- Keeping an optimization with no measurable win (complexity that didn't pay).
- Profiling once. The hotspot moves after each fix; re-profile every round.
- Optimizing without a representative workload (you're tuning to a fiction).
- Grinding past saturation instead of stopping when the wins dry up.

## References

- [ROUTING.md](references/ROUTING.md): profile/optimize/prove/escalate map + staged prompts
