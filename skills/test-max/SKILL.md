---
name: test-max
description: >-
  Drive a test suite to trustworthy, exhaustively, by orchestrating the testing
  skills you own to cover the behaviors that matter, then proving every test earns
  its keep with a mutation gate: plant mutants, and any assertion that lets one live
  is theater and gets fixed. Includes flake-zero, which root-causes nondeterminism
  instead of retrying it. Use when "make the tests trustworthy", "coverage pass",
  "the tests are weak", "can we trust this suite", "kill the flaky tests", a
  pre-ship test gate, or when project-autopilot's testing lens fires.
---

# Test Max

The fourth quality vertical (its siblings: **bug-zero** = correct, **perf-max** =
fast, **ux-maximizer** = beautiful; test-max = *trusted*). Point it at a suite and
drive it to the state where a green run actually means something. It doesn't invent
testing techniques. It **conducts the testing skills you already have** as gap-fillers,
and adds the trust layer they lack: coverage counted over behaviors instead of lines,
every assertion proven lethal against planted mutants, and flakes root-caused instead
of retried.

> **Core Insight:** Line coverage lies. A suite can execute 95% of lines and assert
> almost nothing: the test *ran* the code; it didn't *check* it. The only honest
> measure of a test is whether it kills the bug it claims to guard against, and you
> can measure that directly: plant the bug (a mutant) and watch. An assertion that a
> mutant survives is **theater**. It costs CI time and buys confidence it doesn't
> deliver. Like its siblings, this is a successive-prompter with a **green ratchet**;
> its distinct gate is the mutation-proof.

> **Core Insight #2:** Coverage is a claim about BEHAVIORS, not lines. The inventory
> comes from the project's intent (what must this thing do, what must it never do,
> what does it promise at its boundaries), and each behavior is TESTED or it isn't.
> 100% of lines with the refund-path behavior untested is a worse suite than 60% of
> lines with every promised behavior pinned. Lines are a *hint* for finding untested
> behaviors, never the score.

## What test-max adds (the only new logic)

1. **Behavior inventory:** coverage measured against a ranked list of behaviors
   that matter (from intent + blast radius), not a line percentage.
2. **The mutation-proof gate:** mutants planted in covered code; a surviving mutant
   convicts the covering test of theater. Fix the assertion or write the missing test;
   "covered" is not "checked" until the mutants die.
3. **flake-zero:** nondeterminism is a bug with a root cause, not a weather
   condition. Reproduce it under amplification, fix THE race, prove stability over K
   green runs. Retries are forbidden as fixes.
4. **Saturation convergence:** stop when fresh mutation rounds keep coming up
   lethal (new mutants keep dying) AND the behavior inventory has no untested entry
   above the bar. Not a coverage number, not a counter.

## Preconditions / guards

1. **Green baseline + checkpoint.** A red suite at launch is an escalation: triage to
   green (or get the operator's call on what green means) before strengthening anything.
2. **An intent source exists** (README, plan doc, INTENT.md). The behavior
   inventory derives from it. No intent → run reality-check's intent extraction
   first; you can't rank behaviors you can't name.
3. **Never weaken a test to reach green.** Deleting or loosening an assertion to
   make a run pass is the inverse of this skill's entire job.
4. **Test code follows no-slop discipline.** Strengthen the assertions the gap
   names; no drive-by rewrites of the suite's structure "while you're here".

## The Pipeline (staged: issue each step only when the prior finishes)

```
Phase 0  ORIENT     Read intent + code; write the behavior inventory to
                    <target>/.testmax/BEHAVIORS.md — one line per behavior that
                    matters, ranked by blast radius (data loss > money > auth >
                    correctness > cosmetics). Green baseline, checkpoint.
Phase 1  MAP        Run the suite with line coverage AS A HINT. Mark each inventory
                    entry TESTED (a test asserts it) / EXECUTED-ONLY (lines run,
                    nothing asserts it) / UNTESTED. EXECUTED-ONLY is the theater
                    watchlist for Phase 3.
Phase 2  GENERATE   Route each UNTESTED behavior to the best-fit testing skill
                    (references/MUTATION.md has the map: property/fuzz for parsers
                    and invariants, golden for output stability, metamorphic for
                    relations, real-service-e2e for money/data paths, webapp-e2e for
                    user flows). Batches of ≤5 under the green ratchet.
Phase 3  MUTATE     Plant mutants in covered code (recipes + tools in
                    references/MUTATION.md). Every SURVIVOR is a conviction:
                    the covering test is theater → strengthen the assertion or add
                    the killing test, re-plant, watch it die. (A survivor in dead
                    code → inbox item, never a drive-by deletion.)
Phase 4  FLAKE-ZERO Any test that failed inconsistently during Phases 1–3 (or was
                    already known flaky) goes through the sub-mode below. A flaky
                    test is a lying witness; the suite isn't trustworthy while one
                    remains.
Phase 5  RE-MUTATE  Fresh mutation round on everything touched. Stop when K=2
                    consecutive rounds come up lethal (every new mutant killed) AND
                    the inventory holds no UNTESTED behavior above the agreed bar.
                    Else → back to Phase 2/3 on what survived.
```

## flake-zero (the sub-mode: retries are forbidden as fixes)

A retry is a bug with a snooze button. The sub-mode is no-slop-bug-hunter applied to
nondeterminism:

1. **Reproduce under amplification.** A flake that won't flake on demand isn't
   understood. Amplify along the three axes until it triggers reliably: **loop** (run
   the test N=100+ times), **parallelism** (full suite concurrently; shuffle test
   order), **clock/timing skew** (perturb sleeps, inject delays at awaits/IO, run
   under load). Record which axis triggers it; that names the bug class.
2. **Root-cause THE race:** shared state between tests, order dependence, real-time
   assumptions, unawaited async, port/tmpfile collisions, real-network reliance.
   Route hangs and interleavings to deadlock-finder-and-fixer (if installed;
   otherwise attach a debugger and dump all thread stacks); use trace-it if the
   bad interleaving needs to be made visible first. One sentence names the cause.
3. **Fix THE race, no-slop.** The fix is in the test's (or code's) broken invariant,
   not `retry(3)`, not a bigger sleep, not `@flaky`, not quarantine-forever.
4. **Prove stability:** the fixed test green over K consecutive amplified runs
   (K≥50 looped for the single test, ≥3 full parallel suite runs). Below K, it's
   not fixed; it's resting.

## Contract (what orchestrators can rely on)

- **ship-it** calls this as a pre-ship gate: Phase 0's go/no-go can require a
  test-max pass so "tests green" means "tests trustworthy" before the buttons.
- **project-autopilot:** this is its testing lens. "Tests exist but prove little" /
  "flaky suite" routes here, exactly as the correctness lens routes to bug-zero.
- **bug-zero** hands its regression tests here for mutation-proofing when the fix
  batch is done; test-max hands confirmed product bugs (not test bugs) back to
  bug-zero rather than fixing them itself.
- **Out:** `.testmax/BEHAVIORS.md` (the inventory with final TESTED marks),
  mutation kill-rate per round, the flake ledger (each: root cause sentence + K-run
  proof), green exit always; last action is verify + checkpoint.

## THE EXACT PROMPT

```
/test-max
Target: <project path or subsystem>
Scope: <whole suite | a subsystem | flake-zero only>
Bar: <optional — which inventory ranks must be TESTED; default: all above cosmetics>
Mode: solo (default) | swarm (optional — needs ntm installed)
```

## Anti-Patterns

- Chasing a line-coverage number: the cardinal sin. You'll generate executed-only
  theater and the mutants will convict it anyway.
- Fixing a flake with a retry, a longer sleep, or `@flaky`. That's a snooze button
  on a lying witness.
- Weakening or deleting an assertion to make a mutant round or a flaky run pass.
- Writing tests for behaviors nobody ranked. Inventory first; effort follows blast
  radius, not whichever file is open.
- Treating a surviving mutant in dead code as a testing gap. It's a dead-code
  finding; inbox it, don't test it and don't drive-by delete it.
- Fixing product bugs discovered mid-pass in the same batch. Hand them to
  bug-zero / no-slop-bug-hunter; this pass changes tests, not product behavior.
- Grinding mutation rounds past saturation. Two lethal rounds and a clean
  inventory is done; an infinite mutant farm is not the goal.

## References

- [MUTATION.md](references/MUTATION.md): mutant recipes + per-language tools,
  kill-rate scoring, the behavior→testing-skill map, and the flake amplification
  matrix
