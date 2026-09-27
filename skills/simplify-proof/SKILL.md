---
name: simplify-proof
description: >-
  Reduce code complexity under a behavior lock: a characterization test written
  FIRST pins current behavior (warts included), then refactor one transform at a
  time, prove observably identical, and measure the complexity drop, reverting
  any refactor that didn't measurably simplify (cyclomatic / nesting / length).
  Use when "simplify this", "this module is unreadable", "refactor without
  breaking it", "pay down the complexity", untangling a god-function, or when
  bug-zero / optimize-proof flag complexity as the real problem.
---

# Simplify Proof

optimize-proof's sibling, same spine, inverted objective: optimize-proof buys speed
and proves behavior didn't change; simplify-proof buys readability and proves the
same thing. Both write the proof **before** the change, both measure the claimed
win, and both revert anything that didn't pay. There the price of failure is
complexity that bought no speed; here it's churn that bought no simplicity.

> **Core Insight:** "Refactored" is a two-part claim, exactly like "faster", and
> most refactors prove neither part: that the code still does the *same thing*, and
> that it's *actually simpler*. The proof (a characterization test pinning current
> behavior, **warts included**) must be written FIRST, against the original. A
> characterization written after the refactor pins the new code's behavior and
> blesses whatever the refactor broke. And the warts matter: characterization pins
> what the code DOES, not what it should do. A wart discovered while characterizing
> is a decision-inbox item, never a silent "fix" folded into the refactor. The
> moment the refactor intentionally changes behavior, the proof can't tell you
> about the unintentional changes riding along.

> **Core Insight #2:** Simpler is a measurement, not a feeling. Cyclomatic
> complexity, nesting depth, and length, before and after. Mirroring
> optimize-proof's no-win rule, **a refactor that doesn't measurably drop them
> reverts.** Code that was reshuffled but measures the same didn't get simpler; it
> got different, and different has a review cost with no payoff. "It feels cleaner"
> is the readability version of an unmeasured speedup.

## Honest overlap

The arsenal has an isomorphic-refactor skill for the *mechanics* of equivalence-
preserving transforms. This skill is the **gate** around any such transform:
proof-first, one transform at a time, measured-drop-or-revert. Use its transforms
inside step (d) when it's installed; the gate stays either way.

## Preconditions / guards

1. **A target list with a reason each:** from bug-zero's cleanup route ("the bug
   was really complexity"), optimize-proof's worth-it gate ("this hotspot's fix
   needs the complexity paid down first"), a hotspot-of-confusion the operator names, or
   the complexity scan in references/COMPLEXITY.md. No target → no pass; "simplify
   the codebase" by vibe is refactoring's version of optimizing by vibe.
2. **Settled code only** (the maturity guard, shared with optimize-proof): features
   still churning in the target (>10 modified files OR >3 in-flight beads touching
   it) → STOP. Simplifying code about to be rewritten is pre-paying rent on a
   building being demolished.
3. **Behavior freeze for the whole pass:** no features, no bug fixes in these
   commits. Warts and bugs found en route go to the inbox (route real bugs to
   no-slop-bug-hunter as a SEPARATE task).
4. **Green baseline + checkpoint** before any change. Public API changes are
   ask-user buttons: callers outside the target make "observably identical"
   a promise to strangers.

## The Pipeline (staged: one target fully banked before the next)

### Phase 0: Arm

Read the target list; confirm each entry has a reason, not just a filename.
Green baseline, checkpoint, create `<target>/.simplify/SIMPLOG.md`. Measure each
target's complexity baseline once (references/COMPLEXITY.md) so the worth-it gate
has numbers.

### Per target: the loop body

a. **Understand.** Read the code and its callers. Name *why* it's complex in one
   sentence: tangled state / deep nesting / duplication / wrong abstraction /
   dead paths / doing N jobs. The sentence picks the transform; no sentence → you
   don't understand it enough to change it safely.

b. **Characterize FIRST.** Pin observable behavior with optimize-proof's proof
   recipes. [../optimize-proof/references/PROOFS.md](../optimize-proof/references/PROOFS.md)
   applies verbatim (golden on real inputs, invariants, differential with the
   original kept alive; differential is the natural fit here). Warts included:
   pin the off-by-one, pin the weird error string. Run it green on the ORIGINAL.
   Wart found → inbox, keep pinning.

c. **Baseline.** Record the target's numbers in SIMPLOG: cyclomatic (sum + max),
   max nesting depth, LOC, longest function. These are the "before".

d. **Simplify.** ONE transform at a time, in payoff order: **(1) delete dead
   paths** (the only transform that's pure win) **→ (2) collapse duplication →
   (3) flatten nesting** (guard clauses, early returns) **→ (4) extract/inline to
   the right altitude** (split the god-function; inline the indirection nobody
   needed) **→ (5) rename**: last, riding on a structural change or not at all.

e. **Prove + measure.** Characterization green AND the numbers measurably down.
   Three outcomes: proof red → 2 repair attempts, then REVERT to checkpoint,
   log `reverted: behavior` (a prettier module that does something different is
   a bug with good posture). Proof green but numbers flat or traded sideways →
   REVERT, log `reverted: no-drop` (see COMPLEXITY.md for what counts as a drop;
   LOC alone can be gamed by code golf, which is why the metrics travel together).
   Both hold → bank.

f. **Bank.** Checkpoint commit. SIMPLOG entry: target, why-complex sentence,
   transform, before/after numbers, proof file, status. If the differential
   original was kept alive, delete it NOW: banked means the old copy goes.

g. **Worth-it gate.** Next target: expected drop × how often humans read/change
   this code, versus churn risk. Cold code that's ugly but correct and never
   touched → exit `not-worth-it` with the list intact. The goal is a codebase
   people can change safely, not a beauty contest.

### Phase Ω: Report

Final verify green, final checkpoint. Report: targets banked, cumulative
before/after numbers, everything reverted (and why), inbox items (warts, API
change requests), exit reason (`list-done` / `not-worth-it` / `budget-hit`).
SIMPLOG.md is the audit trail.

## Contract (what orchestrators can rely on)

- **bug-zero**'s cleanup route ("the bug was really complexity") lands here;
  confirmed bugs found here route back to no-slop-bug-hunter. The firewall runs
  both directions.
- **optimize-proof** hands over when a hotspot's fix price is complexity worth
  paying down first, and this skill returns the favor: a simplification that
  might cost performance on a KNOWN hotspot gets a before/after measurement on
  that workload too (both siblings' gates apply on shared ground).
- **project-autopilot**'s maintainability lens routes here for the exhaustive
  treatment; **deepen**-wrappable by design: SIMPLOG plays PASSLOG; banked
  targets are settled, never re-litigated.
- **One characterization test per banked target, committed with it.** The
  refactor and its proof travel together; reverting one reverts both.
- **Green exit, always.** Honest exit status: `list-done` / `not-worth-it` /
  `budget-hit`.

## THE EXACT PROMPT

```
/simplify-proof
Target:  <project path or module>
Targets: <the list with reasons — from bug-zero/optimize-proof handoff, the operator,
          or a COMPLEXITY.md scan>
Budget:  <optional — max targets this run; default: until not-worth-it>
```

## Anti-Patterns

- Characterizing after the refactor: the cardinal sin. It pins the new behavior
  and blesses the breakage.
- "Fixing" a wart mid-characterization because it's obviously wrong. Obviously
  wrong is an inbox item; the proof pins what IS.
- Batching transforms. When the proof reds you can't tell which one broke it;
  one transform, one proof run, one measurement.
- Keeping a refactor with flat numbers because it "feels cleaner." Revert;
  different isn't simpler.
- Code-golfing LOC down while cyclomatic/nesting hold or rise. Density isn't
  simplicity; the co-measured metrics exist to catch exactly this.
- Escalating a refactor into a rewrite ("this whole module should just be…").
  A rewrite is a different conversation with different scope; inbox it.
- Simplifying mid-churn, or grinding cold never-touched code past the worth-it
  gate.
- Renaming as the first transform. Names ride on structure; a rename-only diff
  is review noise with a proof attached.
- Leaving the differential's kept-alive original in the tree after banking.

## References

- [COMPLEXITY.md](references/COMPLEXITY.md): the metrics, per-language tools,
  what counts as a measured drop, and the gaming guards
- [../optimize-proof/references/PROOFS.md](../optimize-proof/references/PROOFS.md):
  the shared proof recipes (golden / invariant / differential) and logging
  discipline; step (b) uses them verbatim
