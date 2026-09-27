# Mutation Proofs & Flake Amplification

Companion to test-max. Three jobs: plant mutants that convict theater, route
untested behaviors to the right generator, and amplify flakes until they confess.

## Part 1: Planting mutants

A mutant is a deliberate, single, plausible bug inserted into code the suite
claims to cover. Run the suite against the mutant: it fails → the mutant is
**killed** (the test is real). It passes → the mutant **survives** and the
covering test is convicted of theater.

### Tools first (use one when the language has one)

| Language | Tool |
|---|---|
| Python | mutmut |
| Rust | cargo-mutants |
| JS/TS | Stryker |
| Go | go-mutesting / gremlins |
| Java/Kotlin | PIT |

Run scoped to the files under test (a whole-repo mutation run on a large codebase
is a weekend, not a phase). Budget per round: the top-ranked inventory behaviors'
modules first; expand only while survivors keep appearing.

### Manual mutants (no tool, or spot-checking a specific suspect test)

Plant by hand, one at a time, always reverted after the verdict:

- **Conditional flip:** `<` → `<=`, `==` → `!=`, negate a guard.
- **Boundary nudge:** off-by-one on a loop bound, index, or pagination limit.
- **Statement drop:** delete a side-effecting line (the write, the publish, the
  cache invalidation). The classic executed-only killer.
- **Return corruption:** return a constant / empty / null instead of the computed
  value.
- **Error-path swallow:** replace a `raise`/`return err` with a silent success.
- **Wrong-variable swap:** substitute a same-typed sibling (`start` for `end`,
  `tenant_a` for `tenant_b`).

Pick mutants that mimic bugs this code could plausibly have. A mutant nobody
would ever write proves nothing when killed.

### Reading the verdicts

- **Killed:** banked. Log the kill; nothing to do.
- **Survived, code is live:** the covering test is theater. Strengthen the
  assertion (assert the value, not just "no exception"; assert the side effect,
  not just the return) or write the missing test. Re-plant the SAME mutant and
  watch it die; the fix isn't banked until the execution.
- **Survived, code is dead/unreachable:** a dead-code finding, not a testing
  gap. Decision-inbox it; never drive-by delete mid-pass.
- **Equivalent mutant** (the mutation genuinely doesn't change observable
  behavior): rare. Log it as equivalent and move on. Don't write a test that
  asserts internals just to kill it; that's inverse theater (a test coupling to
  implementation, not behavior).

### Scoring + saturation

Per round log: mutants planted / killed / survived-theater / survived-dead /
equivalent, in `.testmax/MUTLOG.md`. Saturation = K=2 consecutive rounds with
zero survived-theater across fresh mutants in the ranked modules. Kill rate is a
trend indicator, never a target to game (planting easy mutants inflates it;
plausible bugs only).

## Part 2: Behavior → generator map (Phase 2 routing)

The skills in this table are optional (if installed); without one, write the
test by hand in the style the row names.

| Untested behavior looks like | Route to |
|---|---|
| Parser / serializer / protocol / anything that eats bytes | testing-fuzzing |
| "Output must stay exactly this" (reports, renders, exports) | testing-golden-artifacts |
| A relation that must hold (order-free, merge-consistent, idempotent) | testing-metamorphic |
| Money / data / auth path crossing a real service | testing-real-service-e2e-no-mocks |
| A user-facing web flow | e2e-testing-for-webapps / test-e2e-webapps |
| A spec or protocol the code claims to implement | testing-conformance-harnesses |
| A concurrency invariant | deadlock-finder-and-fixer (as finder) + a pinning test |

Every generated test goes through Part 1 before it counts: a new test that can't
kill its own mutant went in as theater.

## Part 3: Flake amplification matrix (flake-zero step 1)

Amplify one axis at a time; the triggering axis names the bug class:

| Axis | How | Triggers → class |
|---|---|---|
| Loop | run the test N=100+ solo | inherent nondeterminism: unseeded random, map order, real network |
| Parallelism | full suite concurrent; shuffle order; 2× workers | shared state: global fixtures, same DB rows, port/tmpfile collisions, order dependence |
| Clock/timing | inject delays at awaits and IO; run under CPU load; freeze/skew time | races: unawaited async, sleep-based sync, real-time assertions, timeout margins |

If the flake only triggers with parallelism, don't fix it with isolation-by-
serialization (`--workers=1` is a retry in disguise); fix the shared state.
If instrumenting the test makes the flake vanish, that IS evidence (a timing
race): route to deadlock-finder-and-fixer (if installed; otherwise trace-it),
don't declare victory.

Stability proof: K≥50 consecutive amplified green runs for the fixed test,
plus ≥3 full parallel suite runs. Log each fixed flake in `.testmax/FLAKES.md`:
axis, root-cause sentence, fix, K-run proof.
