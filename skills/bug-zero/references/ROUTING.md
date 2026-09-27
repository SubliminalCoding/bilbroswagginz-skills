# Routing: Bug Zero

Which installed skill handles which bug class, plus the staged prompts. bug-zero
adds the sequencing + verification; these skills do the actual finding and proving.
Skills marked † are not part of this pack: use them if installed; otherwise run that
check directly with the smart model and the project's own tests.

## Phase 1 SWEEP: bug-class → finder (fan out, dedupe the union)

| Bug class | Skill |
|---|---|
| General bugs, AI-code validation, pre-commit quality | ubs† / ubs-workflow† |
| Systematic audit→fix→rescan over a surface | bug-sweep |
| Concurrency: deadlocks, races, livelocks, await-holding-lock, DB locks | deadlock-finder-and-fixer† |
| Stubs / mocks / placeholders / fake code in LIVE paths | mock-code-finder† |
| Correctness bugs in the current diff | code-review (built-in) |
| Live process: hangs, spins, segfaults, stuck loops | gdb-for-debugging† |
| Security class: auth bypass, webhook integrity, RLS, secrets | security-sweep |
| Crash discovery in parsers / protocols / serialization | testing-fuzzing† |
| Un-reproducible symptom ("sometimes", "only in prod", heisenbug) | trace-it, upstream of the sweep: capture the bad state FIRST; its located cause joins the findings |

Pick the finders that fit the target (a CLI library skips security-sweep;
a parser adds testing-fuzzing). Run them together; one finder catches what another
is blind to. Dedupe before verifying. (A LIVE production incident routes to
incident-zero instead: stabilize first; it sends the cleanup back here.)

## Phase 2 VERIFY: kill false positives before any fix

- **Smart-model adversarial re-review** (no external skill, just the top model):
  open each finding's cited file/line, confirm the bug is real and reachable.
  Default to "not a bug" if uncertain; a phantom bug fixed is worse than one skipped.
- **multi-model-triangulation**†: for a doubtful or high-stakes finding, cross-check
  against Codex / Gemini / Grok before committing to a fix.
- **lean-formal-feedback-loop**†: for the rare finding worth a formal proof.

## Phase 3–4 PROVE: the fix is real and stays dead

- **verify** (built-in) / **run**: reproduce that the bug no longer happens. Watch
  it not happen; don't trust "tests pass."
- Regression test paired with EVERY fix, chosen by what fits:
  - testing-real-service-e2e-no-mocks†: real DB/API/service flows
  - testing-golden-artifacts†: output-stability regressions
  - testing-metamorphic†: relational/property regressions
  - testing-conformance-harnesses†: spec/protocol compliance
  - testing-fuzzing†: a fuzz harness that guards the fixed parser
  - e2e-testing-for-webapps†: webapp user flows

## Cleanup (optional: when the bug was really something else)

- **simplify / simplify-and-refactor-code-isomorphically**†: the bug was complexity.
- **deps-update / library-updater**†: the bug was a stale or vulnerable dependency.

## Staged prompts

```
Phase 1: sweep the project with [the relevant finders for this target], in parallel.
         collect and dedupe every finding. do not fix anything yet.

Phase 2: now YOU carefully re-review each finding against the actual code. is it
         truly a bug? reachable? which ones are real, which are false positives?
         default to "not a bug" if you're not sure. (triangulate the doubtful ones.)

Phase 3: fix the confirmed bugs one batch at a time, and pair EACH fix with a
         regression test that would have caught it. if a fix breaks any test, revert
         that batch.

Phase 4: reproduce that each fixed bug is actually gone by running it, not just by
         the test suite passing.

Phase 5: re-sweep. keep going until two passes in a row find nothing real.
```
