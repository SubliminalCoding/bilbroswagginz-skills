---
name: deepen
description: >-
  Multi-pass engine: iteratively apply any named skill or slash command over a
  target with progressive deepening: each pass builds on the prior pass's log,
  every pass's findings get FIXED (under the green ratchet) before the next pass
  fires, and the loop stops on saturation, not at an arbitrary count or at zero.
  Use when "apply N times", "keep running it until it stops finding things",
  "multi-pass", "grind this skill over the project", "another pass", or when
  ux-maximizer / project-autopilot route counted refinement passes here.
---

# Deepen

Turn a one-shot skill into a multi-pass grind. Point it at any skill (ux-scan,
polish-pass, a security sweep, optimize-proof, unslop), name a
target and a pass cap, and it runs the skill repeatedly with each pass digging
past the last: prior findings are fixed and off the table, so every pass is
forced into new territory until territory runs out.

> **Core Insight:** N passes of the same prompt is not N× the value; it's 1×
> the value plus N−1 reruns. The value of a repeat comes from three disciplines:
> **deepen** (pass k carries the log of passes 1..k−1, so it cannot re-find old
> ground and must go further), **fix-then-pass** (every pass's findings are fixed
> *before* the next pass fires, so the next pass inspects the improved artifact,
> not a stale one plus a findings pile), and **stop on saturation** (a pass that
> surfaces nothing materially new ends the loop, even with passes left on the
> cap; chasing literal zero manufactures non-findings). This is a
> *successive-prompter*: one pass at a time, each issued only when the prior
> pass's fixes are banked green, never a "do 5 passes" megaprompt.

## Preconditions / guards

1. **The wrapped skill exists and applies.** `/deepen` adds repetition discipline,
   not capability. If one pass of the skill wouldn't help, five won't.
2. **Green baseline + checkpoint.** The ratchet needs a recoverable starting
   point: build + tests + smoke green, checkpoint commit taken, before pass 1.
3. **A verification command.** Know what "green" means for this target (test
   suite, build, smoke) before starting. A ratchet with no test is a hope.
4. **For review/audit-class skills:** the smart-model re-review gate (Phase
   per-pass, step c) is NON-SKIPPABLE; budget for it.

## The Pipeline (staged: issue each pass only when the prior pass is banked)

### Phase 0: Arm
Resolve the four parameters: **Skill** (what to apply), **Target** (where),
**Passes** (the cap: a ceiling, never a quota), **Stop** (the convergence rule;
default saturation). Confirm the green baseline, take the checkpoint, create
`<target>/.deepen/PASSLOG.md`.

### Per pass k (1..cap): the loop body

a. **Brief.** Issue the wrapped skill against the target. Pass 1 runs broad:
   the skill as designed. Pass k ≥ 2 gets the PASSLOG prepended with the
   deepening charge: *everything already found and fixed is settled; do not
   re-litigate it; go deeper* (new categories, edge cases, layers beneath the
   surface fixes, what the prior fixes themselves exposed). Verbatim templates
   in references/PASSES.md.

b. **Collect.** Capture the pass's findings as a numbered list with impact
   (material / minor / nit).

c. **Smart-model re-review gate** (review/audit-class passes; non-skippable,
   standing rule). Fresh top-model eyes on the findings against the
   actual code: is each one real, still-present, and worth fixing? Demote
   hallucinated or already-fixed findings *before* spending fix effort. Skip
   only for skills that change code directly rather than emit findings.

d. **Fix all of it, now.** Every surviving material finding gets fixed in this
   pass, one at a time, before pass k+1 exists. **Green ratchet between fixes:**
   a fix that breaks build/tests/smoke gets 2 repair attempts, then is REVERTED
   to checkpoint and logged `reverted`. A pass's change that breaks a test is
   not kept, however right the finding felt. Checkpoint after the pass ends
   green.

e. **Log + stop-check.** Append the pass entry to PASSLOG.md (findings, fixed,
   reverted, deferred-with-reason). Then apply the Stop rule. Default:
   **saturation, not zero:** the pass produced no *materially* new findings
   relative to effort spent (only nits, rephrases of settled items, or
   manufactured work). Saturated → exit the loop even with passes remaining.
   Cap reached → exit with status `cap-hit` and say so; never silently grind on.

### Phase Ω: Report
Final verify green, final checkpoint, then a short done report: passes run,
exit reason (`saturated` / `cap-hit` / `stop-rule`), the per-pass deltas, and
anything reverted or deferred. PASSLOG.md is the audit trail.

## Contract (what orchestrators can rely on)

ux-maximizer runs its quality cascade through this skill (3× ux-scan, 5×
polish-pass, 3× design-intel: "each with progressive deepening; fix every
pass's findings"), and project-autopilot routes "repeated multi-pass refinement
of one facet" here, bounded by its CONVERGENCE.md saturation gate. The
interface is fixed:

- **Parameters in:** skill/command · target · max passes (cap) · stop condition.
- **`<target>/.deepen/PASSLOG.md`:** per-pass log (format in
  references/PASSES.md): what each pass found, fixed, reverted, deferred. The
  caller reads this to see what changed per pass.
- **Green exit, always:** the loop never hands back a red target; last action
  is verify + checkpoint or revert.
- **Honest exit status:** `saturated` (the good ending) / `cap-hit` (cap was
  the binding constraint; caller may re-arm with a higher cap) / `stop-rule`
  (a caller-supplied condition fired).

## THE EXACT PROMPT

```
/deepen
Skill:   <skill or slash command to apply, e.g. ux-scan>
Target:  <project path or scope, e.g. ~/my-app or "the checkout flow">
Passes:  <max pass cap, e.g. 5 — a ceiling, not a quota>
Stop:    <optional override — default: saturation (no materially new findings)>
```

## Anti-Patterns

- Re-running the identical prompt N times. Without the PASSLOG charge, pass 3
  proudly re-finds pass 1's settled items.
- Collecting findings across passes and "fixing them at the end." Pass k+1
  must inspect the *fixed* artifact, or it audits a ghost.
- Keeping a pass's change that broke a test because the finding was right.
  Revert it; right-but-red is red.
- Treating the cap as a quota. Grinding to N after saturation manufactures
  non-findings (the Convergence clamp exists for exactly this).
- Chasing literal zero. The stop signal is diminishing returns per token, not
  an empty findings list.
- Skipping the smart-model re-review on audit passes because the findings
  "look real." First-draft findings always look real to their author.
- One megaprompt ("do 5 ux-scan passes and fix everything"). Successive
  prompts, one pass at a time, or the deepening discipline collapses.
- Widening scope mid-loop ("while I'm here..."). The loop deepens one facet;
  new facets are a new `/deepen` run or an inbox item.

## References

- [PASSES.md](references/PASSES.md): verbatim pass-1 and pass-k prompt
  templates, the saturation test, and the PASSLOG.md entry format
