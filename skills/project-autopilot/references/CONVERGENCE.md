# Convergence: when the loop is DONE

"Ad infinitum" without a stop condition is churn. The loop runs until the project
satisfies its Intent Statement, defined mechanically:

## The green ratchet (per-tick invariant)

Every tick must END green or be rolled back. Green means:

1. Build/compile passes
2. Test suite passes (no new skips, no deleted assertions to force passing)
3. Smoke check passes (app starts; for webapps the core flow loads; for CLIs
   `--help` + one happy-path invocation)
4. Checkpoint commit created (conventional message; git-commit-craftsman if installed)

A batch that can't reach green after 2 fix attempts gets **reverted to the last
checkpoint** and its gap goes to the decision inbox tagged `failed-auto` with the
error output attached. The loop moves on. No wedged states, ever.

## Done criteria (in order)

1. **Bead drain:** `br ready` is empty AND the decision inbox holds every
   remaining item.
2. **Dry passes:** K = 2 consecutive full diagnosis passes (all relevant lenses)
   emit zero auto-routable gaps with impact ≥ medium.
3. **Final reality check:** run reality-check against the Intent
   Statement. If it surfaces a strategic gap, that's one more cycle (or an inbox
   item if it's an operator decision), then re-check.
4. **Done report:** write the run report (see SKILL.md Phase 6) and STOP.

Low-impact gaps do NOT block done. They get listed in the report under
"declined polish". Silence about dropped work is forbidden; doing it all is not required.

## Maturity pass: the late-stage optimization window

There is one activity that belongs AFTER bead drain (step 1) and before the final
dry passes (step 2), and only here: **the two-step optimization pass**
(OPERATOR-MOVES "Late-stage optimization pass"). Once features have stopped
churning and the core goals are WORKING, orient + `hotspot-scout`
for a scored hotspot list, then `deepen` over
`optimize-proof` on the top hotspots.

Why here and nowhere earlier: profiling a half-built, still-changing codebase
optimizes code that's about to be rewritten: wasted effort that can also lock in
premature structure. The loop must NOT trigger this during active feature build.
The signal that it's time: `br ready` is essentially drained of feature work and
the correctness lens has been green across the last few ticks. Each optimization
stays under the green ratchet. This pass is optional and impact-gated like any
other work. Skip it for a fun side project where speed isn't part of the Intent
Statement; run it for anything where performance is a stated or shipped concern.

## Saturation gate + re-diagnosis sweep (the loop-until-dry engine)

This is the heart of the operator's universal in-flight prompt. After bead drain, the loop
transitions to a **review phase** (in swarm mode: vibing-with-ntm code-review with
cc + cod, if installed; solo: a code-review pass / bug-sweep). Stop a review loop on
**saturation, not on zero**. The signal is *diminishing returns per token*: few new
real bugs found relative to the effort and tokens spent. Chasing literal zero forever
manufactures non-findings (that's what the Convergence clamp prevents).

When review saturates, branch:

- **Suspect no more material work?** → proceed to the done criteria (final reality
  check → done report).
- **Suspect a different CLASS of work remains?** → run a fresh **diagnosis sweep**
  with the named analysis skills, *to the extent each applies* (the non-pack ones only
  if installed): mock-code-finder, deadlock-finder-and-fixer, reality-check,
  modes-of-reasoning-project-analysis,
  hotspot-scout (late-stage only, per the optimization gate),
  security-sweep. Each new finding → smart-model self-review → beads
  (frozen template) → back into execution. This is the loop *re-arming* itself with
  a new lens, not spinning on the old one.

The whole point: the loop generates its own next round of work from a fresh angle
when the current angle is exhausted, then stops cleanly when fresh angles stop
producing material beads (K=2 dry sweeps, per the done criteria above).

## Loop budget (hard backstops, independent of done criteria)

- **Max ticks per launch:** 25 (solo) / 50 (swarm mode). Hitting the cap = write
  the report with status `budget-exhausted`, never silently continue.
- **Max consecutive reverts:** 3 → stop, report `unstable`, recommend human session.
- **No-progress detector:** 3 consecutive ticks with no net diff and no new inbox
  items → the loop is spinning; stop with status `stalled`.
- **Scope ratchet:** the loop may CLOSE gaps, never widen the Intent Statement.
  "While I'm here, it could also..." is exactly the drift this skill exists to
  prevent. New scope ideas go to the inbox as idea-forge candidates.

## Re-entry

The loop is resumable: state lives in beads + the decision inbox file + checkpoint
commits, not in conversation memory. A fresh session re-launches by reading those
three artifacts, never by re-diagnosing from scratch (re-diagnosis happens only
when the bead graph is empty).
