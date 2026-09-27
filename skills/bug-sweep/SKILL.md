---
name: bug-sweep
description: >-
  Systematic audit→fix→rescan over a code surface to find MANY bugs, not one
  reproduced bug. Walks the same surface through seven lenses (logic, boundaries,
  error handling, state/lifecycle, data integrity, concurrency-adjacent, API
  misuse), one lens per pass, dedupes the candidates, verifies each is real before
  touching anything, fixes the survivors no-slop under a green ratchet with a
  regression test each, then re-sweeps until consecutive sweeps come up dry. Use
  when "sweep this for bugs", "find all the bugs in this module", "deep bug
  pass", "comb through this", "audit this surface for defects", or when bug-zero
  routes its systematic audit→fix→rescan class here.
---

# Bug Sweep

The breadth finder. no-slop-bug-hunter kills one reproduced bug; this combs a whole
surface for the population. The trick is that one read of code only finds the bugs
you were primed to see. So it reads the same lines seven times, each pass primed by
a different lens, collects everything, then verifies before it fixes and re-sweeps
after it does. It stops when sweeps come up dry, not when a pass counter runs out.

> **Core Insight:** Bug-finding yield is set by *priming*, not effort. Ask a model
> to "find the bugs" and it finds the three most obvious, declares the file clean,
> and is wrong: the boundary bugs and lifecycle bugs were invisible because nothing
> pointed the read at them. Seven narrow passes over the same surface out-find one
> broad pass every time. And because every finder over-reports, the second half of
> the trick is the firewall between *finding* and *fixing*: a candidate is not a bug
> until it survives an adversarial re-read at its own file:line, and a fix is not
> banked until a test that fails on the buggy code says it's real.

## Preconditions / guards

1. **Bound the surface before pass 1.** Use an explicit file list, plus one line on what
   each file is *supposed* to do (a bug is a divergence from intent; no intent, no
   verdict). "The whole repo" unbounded dilutes every lens; sweep a subsystem at a
   time.
2. **Standalone mode: green baseline + checkpoint.** The ratchet needs a known-good
   start. Suite already red → escalate for a green baseline before any fix. Callee
   mode (called by bug-zero): **read-only, touch nothing**. The orchestrator owns
   verify/fix/prove.
3. **Never weaken tests to reach green.** Deleting/skipping assertions hides bugs.
4. **Top-model work, no qwen variant.** Lens reading is adversarial close-reading,
   which is exactly what small models hallucinate at. Phantom findings poison the whole
   downstream loop.

## The seven lenses (one per pass)

Each pass walks the ENTIRE surface looking for ONE class only. Full telltales, greps,
and the verify question per lens are in [references/LENSES.md](references/LENSES.md):

1. **Logic errors:** inverted/wrong conditions, wrong variable, copy-paste branch
   not edited, unit mismatches, operator slips (`<` vs `<=`, `and` vs `or`).
2. **Boundary / edge cases:** empty, zero, one, negative, max, last iteration,
   off-by-one in slices/pagination, float equality, unicode, midnight/DST.
3. **Error handling:** swallowed exceptions, broad catches, error paths returning a
   shape the caller can't take, missing cleanup on the failure branch, lying exit codes.
4. **State & lifecycle:** stale cache after mutation, init-order dependence, mutable
   default args, request state in singletons, missing reset between iterations,
   double registration, teardown order.
5. **Data integrity:** lossy conversions, partial writes with no atomic rename,
   multi-step mutation with no transaction, round-trip asymmetry, null vs missing-key
   conflation, naive datetimes crossing a boundary.
6. **Concurrency-adjacent:** check-then-act, non-atomic read-modify-write, lazy-init
   races, mutating while iterating. (Shallow shapes only. Anything that smells like a
   real deadlock/race hands off to **deadlock-finder-and-fixer**.)
7. **API misuse:** same-typed args swapped, failure-signaling return values ignored,
   semantics that changed in the version actually pinned, shell-quoting and regex
   pitfalls, locale/timezone-sensitive calls.

## The Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  BOUND      Fix the surface: explicit file list + intent per file (guard #1).
                    Standalone: establish + checkpoint a green baseline.
Phase 1  PASSES     Seven passes, one lens each, over the SAME surface. Collect
                    candidates. DO NOT fix anything mid-pass. Log per-pass yield
                    ("pass 4 state/lifecycle: 3 candidates") so the tail is visible.
Phase 2  DEDUPE     Merge candidates that share a root cause (two lenses hitting one
                    bug = ONE finding carrying both tags). Emit the findings list in
                    the record format below.
                    ── CALLEE MODE ENDS HERE: hand the list to bug-zero. ──
Phase 3  VERIFY     Adversarially re-read EVERY candidate at its cited file:line. Is
                    it real, reachable, a divergence from intent? Or enforced one
                    layer up? Default to "not a bug" if uncertain; a phantom fixed is
                    worse than a bug skipped. Doubtful → multi-model-triangulation.
Phase 4  FIX        Survivors only, batches of ≤5, under the green ratchet, with
                    no-slop-bug-hunter's per-fix contract verbatim: root cause named
                    in ONE sentence, exact scope named, fix THE bug not the category.
                    Each fix ships a regression test that FAILS on the buggy code.
                    A fix that reds any test → revert that batch. In a beads repo:
                    one br issue per confirmed bug, closed by the fixing commit.
Phase 5  PROVE      Run the repro; watch each bug be gone. "Tests pass" isn't proof.
Phase 6  RESCAN     Full re-sweep, ALL seven lenses. Fixes shift the surface, and a
                    lens that was dry last sweep can produce now. DRY = a full sweep
                    finding nothing materially new. Stop at K=2 consecutive dry
                    sweeps (saturation), never at a fixed pass count.
```

## The findings record (the contract)

One block per deduped finding. This exact shape is what bug-zero's VERIFY consumes:

```
- id: cache-invalidate-misses-mirror        # stable slug = cross-finder dedupe key
  lens: [state-lifecycle, data-integrity]   # every lens that hit it
  class: stale cache after mutation
  where: src/cache.py:142
  why: invalidate() clears the local dict but not the redis mirror, so the next
       read within TTL serves the deleted entry
  repro: delete a key, GET it twice inside the TTL window
  severity: wrong-result    # data-loss > crash > wrong-result > degraded > cosmetic
  status: candidate         # candidate | confirmed | phantom | fixed
```

`id` + `where` make merging with another finder's output mechanical; `lens` tags let
the orchestrator see which reading found what; `why` is one sentence or it isn't done.

## Contract (what orchestrators rely on)

- **bug-zero** routes "Systematic audit→fix→rescan over a surface" here in its Phase 1
  SWEEP. Callee mode: run Phases 0–2 read-only and return the deduped, lens-tagged
  candidate list into bug-zero's Phase 2 VERIFY. From there bug-zero owns sequencing,
  the verify loop, the ratchet, and proof. Do not fix, do not verify, do not dedupe
  against other finders (bug-zero merges the union).
- **Standalone** owns the full loop end-to-end, Phases 0–6.
- **Division of labor with no-slop-bug-hunter:** one reproduced symptom → that skill;
  "how many bugs are in here" → this one. This skill's Phase 4 *is* that skill's
  contract applied per survivor.

## Skills it composes with

**no-slop-bug-hunter**: the per-fix discipline Phase 4 inherits. **bug-zero**: the
orchestrator that calls it and merges it with the other finders.
**multi-model-triangulation**: doubtful or high-stakes candidates before any fix.
**deadlock-finder-and-fixer**: handoff target when lens 6 smells a real race.
**verify / run**: Phase 5 proof. **beads-br / beads-bv**: confirmed-bug tracking and
triage when the surface lives in a beads repo.

## THE EXACT PROMPT

```
/bug-sweep
Surface: <files / module / subsystem — explicit, bounded>
Mode:    standalone | findings-only (bug-zero callee)
Lenses:  all (default) | <subset, e.g. error-handling,state-lifecycle>
```

## Anti-Patterns

- One broad "find the bugs" pass instead of lens-per-pass. You find only what you
  were primed for and declare clean too early.
- Fixing mid-pass. Finding and fixing in the same breath narrows the read; collect
  first, fix in Phase 4.
- Fixing a candidate before it survives Phase 3. This is the cardinal sin; every finder
  over-reports, including this one.
- Grep-and-patch off one finding. Sibling instances of the pattern ARE in scope for
  a sweep, but each is its own finding, verified independently at its own line.
- A fixed pass count or stopping at literal zero. DRY means K=2 consecutive full
  sweeps with nothing materially new.
- Dropping a lens because its last pass was dry. Fixes shift the surface; the rescan
  runs all seven, every time.
- Drive-by refactors, symptom-swallowing try/except, feature flags as fixes. The
  no-slop contract applies to every fix in the batch.
- Banking a fix whose regression test doesn't fail on the buggy code.
- Callee mode touching the working tree. Findings go up; hands stay off.
- Running the passes on a small local model. Phantom candidates flood the verify
  loop and burn the saved tokens tenfold.

## References

- [LENSES.md](references/LENSES.md): the seven lenses expanded. What each pass reads
  for, telltale shapes and greps, and the verify question that decides real vs phantom.
