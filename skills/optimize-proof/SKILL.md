---
name: optimize-proof
description: >-
  Execution half of the performance two-step: take a profiler-SCORED hotspot list
  and convert the top targets into measured, behavior-proven speedups: one target
  at a time, proof harness written before the optimization, before/after measured
  on the real workload, green ratchet on every change, algorithmic wins before
  micro-tuning, stop when the next hotspot isn't worth the complexity. Use when
  "optimize this hotspot", "grind the hotspot list", "make this faster without
  breaking it", or when perf-max / project-autopilot hand over "scored hotspot
  list ready to optimize".
---

# Optimize Proof

Step 2 of the pack's performance two-step: **hotspot-scout** finds and scores the
bottlenecks; this skill kills them. Feed it the scored list and it works the top
targets one at a time, each change measured before/after on the real workload,
each change backed by a behavior proof (a test demonstrating identical observable
behavior), each change banked or reverted under the green ratchet. It never picks
its own targets; the profiler already did.

> **Core Insight:** "Faster" is a two-part claim and most optimization work proves
> neither part: that the change sped up the *real workload* (not a micro-benchmark
> fiction), and that the program still does the *same thing*. This skill makes both
> non-negotiable per change, and the proof is written **before** the optimization,
> against the original code. A proof written after can only encode what the new
> code does; if the optimization quietly changed behavior, the after-proof blesses
> the bug. Fast and wrong is just wrong with better latency.

> **Core Insight #2:** The win ladder is steep: a better algorithm or data
> structure beats not-doing-the-work (cache / batch / lazy), which beats
> doing-less-per-item, which beats micro-tuning, usually by an order of magnitude
> per rung. Spend the first thinking on *why* the target is hot (its complexity
> class, its redundant work, its I/O shape) and attack the highest rung that
> applies. Micro-tuning the inner loop of an O(n²) scan is polishing the wrong
> thing. And every rung has a price in complexity. When the next hotspot's
> plausible win no longer covers that price, stop. A codebase nobody can read is
> its own performance bug.

## Preconditions / guards

1. **A scored hotspot list exists**, from hotspot-scout (or an equivalent
   profiler-scored list: rank, location, dimension CPU/mem/I/O/contention,
   evidence). No list → STOP and route to hotspot-scout first. Optimizing by
   vibe is the cardinal sin; the profiler picks the work, never intuition.
2. **Not during active feature build** (project-autopilot's baked-in anti-move).
   Features still churning (>10 modified files OR >3 in-flight beads) → STOP.
   Optimization is a maturity activity; don't tune code about to be rewritten.
3. **A reproducible real workload**: the same realistic run hotspot-scout
   profiled. If the workload can't be re-run on demand, there is no
   before/after and no skill.
4. **Green baseline + checkpoint** before any change. The ratchet needs a
   recoverable starting point and a known meaning of "green".
5. **Heavy local runs go through memguard** on a unified-memory box (e.g. an
   NVIDIA DGX Spark/GB10 class machine): profiled workloads, model-backed benchmarks,
   anything memory-hungry launches as `memguard run --profile perf -- CMD`,
   never bare. A measurement run that freezes the box measures nothing. On
   ordinary discrete-GPU or laptop hardware, run directly.

## The Pipeline (staged: one target fully banked before the next exists)

### Phase 0: Arm
Read the scored list; confirm each top entry has evidence, not just a name.
Re-measure the #1 target's baseline on the real workload. If the list's numbers
don't reproduce, the list is stale: re-scout, don't optimize a ghost. Confirm
green baseline, checkpoint, create `<target>/.perf/OPTLOG.md`.

### Per target (top unworked entry on the list): the loop body

a. **Understand.** Read the hot code and its callers. Name *why* it's hot in one
   sentence (complexity class / redundant work / I/O shape / contention). That
   sentence picks the ladder rung; no sentence → you're not ready to change it.

b. **Proof first.** Before touching the code, write the behavior proof: a test
   that pins the target's observable behavior (golden outputs on representative
   inputs (edge cases included), invariants, metamorphic relations where exact
   goldens are brittle), **with detailed logging** so a future red shows *what*
   diverged, not just that something did. Run it green against the unoptimized
   code. Recipes in [references/PROOFS.md](references/PROOFS.md).

c. **Baseline.** Measure before: N runs of the real workload (N≥5 for sub-second
   paths, ≥3 for long ones), record median + spread in OPTLOG. A delta inside
   the spread is noise, not a win. Know the noise floor before claiming above it.

d. **Optimize.** One change at a time, highest applicable rung first:
   **(1) algorithm/data structure → (2) don't do the work** (cache, dedupe,
   batch, precompute, lazy) **→ (3) do less per item → (4) move it off the hot
   path** (async, background, ahead-of-time) **→ (5) micro-tune**: last, and
   only with the upper rungs exhausted.

e. **Prove + measure.** Behavior proof green AND after-numbers on the *same*
   workload, same N. Three outcomes: proof red → 2 repair attempts, then REVERT
   to checkpoint and log `reverted: behavior` (right-but-red is red). Proof green
   but the delta is inside the noise floor → REVERT and log `reverted: no-win`
   (don't keep complexity that didn't pay). Both green → bank.

f. **Bank.** Checkpoint commit. OPTLOG entry: target, why-hot sentence, ladder
   rung, before/after numbers, proof file, status. Mark the entry worked.

g. **Worth-it gate.** Look at the next unworked entry: score × plausible speedup
   vs. complexity of the likely fix. Not worth it → exit `not-worth-it` with the
   remaining list intact. The goal is a fast program, not an empty list.

### Phase Ω: Report
Final verify green, final checkpoint. Report: targets worked, cumulative
before/after on the workload, anything reverted (and why), exit reason
(`list-done` / `not-worth-it` / `budget-hit`). OPTLOG.md is the audit trail.

## Contract (what orchestrators can rely on)

perf-max routes "scored hotspot list ready to optimize" here. Its documented
two-step is *Step 1 profile (hotspot-scout) → Step 2 wrap THIS skill in
`deepen` and grind the top hotspots one at a time, each behavior-proofed and
green-ratcheted*. project-autopilot routes the same trigger here automatically,
with the anti-move baked in: never during active feature build, never without a
profiler-scored target list. The interface is fixed:

- **In:** scored hotspot list (hotspot-scout output or equivalent) · target ·
  reproducible workload · optional target budget.
- **`<target>/.perf/OPTLOG.md`:** per-target log: why-hot, rung, before/after
  numbers, proof reference, banked/reverted. The caller reads this for deltas.
- **One proof test per banked change**, committed with it. The speedup and its
  proof travel together; reverting one reverts both.
- **Green exit, always:** last action is verify + checkpoint or revert.
- **Honest exit status:** `list-done` / `not-worth-it` (the good endings) /
  `budget-hit` (caller may re-arm).
- **Deepen-wrappable by design:** one target per pass; OPTLOG plays the role of
  deepen's PASSLOG (banked targets are settled; the next pass starts at the
  next unworked entry, never re-litigates).

## THE EXACT PROMPT

```
/optimize-proof
Target:   <project path or hot subsystem>
Hotspots: <scored list path, e.g. <target>/.perf/HOTSPOTS.md from hotspot-scout>
Workload: <the reproducible real workload to measure against>
Budget:   <optional: max targets this run; default: until not-worth-it>
```

## Anti-Patterns

- Optimizing without a scored list: picking targets by intuition, or "while I'm
  in this file anyway". The profiler picks; you execute.
- Writing the behavior proof *after* the optimization. It pins the new behavior
  and blesses whatever the change broke.
- Measuring a micro-benchmark instead of the real workload. You'll tune for a
  fiction and ship the same latency.
- Single-run before/after numbers: a delta inside run-to-run noise is not a win.
- Keeping a speedup that reds the proof because the numbers were great. Revert;
  fast-and-wrong is wrong.
- Keeping a no-win change because it "should help at scale". No measured gain,
  no merge; complexity must pay rent now.
- Micro-tuning rung 5 while a rung-1 algorithmic fix is available. Read the
  why-hot sentence again.
- Batching several optimizations into one change. When the proof reds, you
  can't tell which one broke it; one change, one proof, one measurement.
- Launching heavy measurement runs bare on a unified-memory box that has
  memguard. Wrap it or it didn't happen (a frozen box mid-benchmark loses the
  session AND the numbers).
- Grinding the list to empty past the worth-it gate. An empty list is not the
  goal; a fast program is.

## References

- [PROOFS.md](references/PROOFS.md): behavior-proof recipes (golden, invariant,
  metamorphic), the logging discipline, and the measurement protocol
