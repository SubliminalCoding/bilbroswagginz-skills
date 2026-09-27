# Lenses: Bug Sweep

One lens per pass, the whole surface per lens. Each entry: what the pass reads for,
the telltale shapes (and greps where grep helps; most of these are read-only finds),
and **the verify question**: the single question Phase 3 must answer YES to before
the candidate is allowed to become a finding. If the answer is "probably", it's a
phantom. Drop it.

## 1. Logic errors

Read every conditional and every assignment as if it's wrong, and prove it right.

- Inverted conditions (`if not valid` guarding the *valid* path), `<` vs `<=`,
  `and` vs `or`, `is` vs `==` on values.
- Copy-paste pairs where one branch wasn't edited (handle-X / handle-Y bodies that
  differ in one identifier, or don't when they should).
- Wrong variable: shadowed names, loop variable used after the loop, `x` where the
  sibling `y` was meant (especially same-typed coordinate/min-max/src-dst pairs).
- Unit and scale mismatches: ms vs s, bytes vs KB, 0-based vs 1-based, degrees vs
  radians: anywhere two numeric domains meet.
- Boolean-returning calls whose result is computed and discarded.

**Verify question:** can you name the input for which the code's answer differs from
the intent's answer? No concrete diverging input → not a finding.

## 2. Boundary / edge cases

Read every loop, slice, and size assumption at its extremes.

- Empty collection, single element, exactly-at-capacity, one-past-capacity.
- Zero, negative, and max for every numeric input; integer truncation on division.
- First and last iteration: fencepost slices (`[:n]` vs `[:n+1]`), pagination that
  drops or doubles the boundary page, `range` ends.
- Float equality and accumulation; NaN flowing through comparisons.
- Strings: empty, whitespace-only, multibyte/emoji length vs byte length, the
  separator appearing inside a field.
- Time: midnight, month/year rollover, DST transitions, naive vs aware comparison.

**Verify question:** trace the exact boundary value through the code by hand. Does
it actually misbehave, or does an upstream guard already exclude it?

## 3. Error handling

Read every `except`/`catch`/`if err` and every early return on the failure branch.

- Swallowed exceptions: bare/broad catches that `pass`, log-and-continue where the
  caller assumes success, `catch (e) {}`.
- Failure branch returns a shape the caller can't take: `None` where the caller
  indexes, `[]` where the caller distinguishes "empty" from "failed", `-1` consumed
  as a valid index.
- Missing cleanup on the failure path: file handles, temp files, locks, transactions
  left open when the happy-path `close()` is skipped by an early raise.
- Error messages that themselves throw (formatting a variable that's `None` exactly
  when the error fires).
- Retries with no idempotency on the retried operation; exit codes that report 0
  after a logged failure.

**Verify question:** name the trigger that takes the failure branch *in this call
graph*. An unreachable failure branch that's wrong is cosmetic, not a bug.

## 4. State & lifecycle

Read every piece of state for who writes it, who reads it, and what resets it.

- Stale derived state: a cache/index/aggregate not invalidated on one of the
  mutation paths (count the mutation sites; count the invalidation sites; compare).
- Initialization order: module A reads config module B sets, import order decides.
- Mutable default arguments; class-level mutable attributes shared across instances.
- Request/session state on singletons or module globals in a server context.
- Missing reset between iterations of a reuse loop (accumulators, buffers, flags).
- Double registration (event handlers, signal connects, atexit) on re-entry paths;
  teardown order using a thing already torn down.

**Verify question:** name the two operations, in order, that leave the state wrong
("mutate via path B, then read"), and confirm no third thing resets it in between.

## 5. Data integrity

Read every place data changes representation or crosses a durability boundary.

- Lossy conversions: int truncation, encoding round-trips, precision-dropping
  serialization (float → JSON → float is fine; Decimal → float is not).
- Partial writes: file written in place with no atomic temp-file + rename; a crash
  mid-write leaves a corrupt artifact the next run trusts.
- Multi-step mutations with no transaction: row A updated, then a failure, row B
  never written, invariant between them now false.
- Round-trip asymmetry: `serialize(deserialize(x)) != x` for some field (defaults
  injected, key order significant, None dropped).
- Null vs missing-key conflation in dicts/JSON/DB columns where the two mean
  different things.
- Naive datetimes persisted, compared, or arithmetic'd across a timezone boundary.

**Verify question:** what concrete artifact ends up wrong on disk / in the DB / on
the wire, and what later read trusts it?

## 6. Concurrency-adjacent

Read for the shallow shapes a sequential read can catch. This lens is NOT a race
detector. Anything needing schedule reasoning hands off to
**deadlock-finder-and-fixer** as its own task.

- Check-then-act: `if not exists: create` on files, rows, keys; anything another
  actor can win between the check and the act.
- Non-atomic read-modify-write on shared state: `count = get(); set(count+1)`
  against a DB/cache/file that more than one process touches.
- Lazy initialization reachable from two entry points with no guard.
- Mutating a collection while iterating it (directly, or via a callback invoked
  mid-loop).
- A file used as a lock without `O_EXCL`/`flock`; PID files trusted without
  liveness checks.

**Verify question:** is there actually more than one actor? A check-then-act in a
strictly single-process CLI with no concurrent invocation is a non-finding. Say so
and move on.

## 7. API misuse

Read every third-party and stdlib call against what the API actually promises, in
the version this project pins, not the version you remember.

- Same-typed arguments swapped (`copy(dst, src)` shapes, (lat, lon) vs (lon, lat),
  start/end pairs).
- Return values that signal failure ignored: `write()` partial counts, `Popen`
  return codes, functions returning a new object the caller expected mutation from
  (`sorted` vs `.sort()`, `df.drop` without `inplace`/reassignment).
- Semantics changed across the pinned version: check the lockfile, then the
  changelog, before calling current behavior a bug (or remembered behavior correct).
- Subprocess/shell: `shell=True` with interpolated input, args needing quoting,
  PATH assumptions.
- Regex: unanchored matches used as full-string validation, `.` matching where `\.`
  was meant, greedy quantifiers crossing delimiters.
- Locale/timezone-sensitive calls (`strftime`, case-folding, number parsing) where
  the output feeds a parser or a comparison.

**Verify question:** open the docs/source for the PINNED version and quote the line
that says the call doesn't do what this code needs. No quote, no finding.

## Pass mechanics (applies to every lens)

- Walk the **entire bounded surface** per pass. Don't sample.
- Record candidates immediately in the findings-record format (SKILL.md) with
  `status: candidate`. Never fix mid-pass.
- Log the per-pass yield line even when it's zero. The yield log is how the rescan
  knows which lenses still have a tail and how saturation gets judged honestly.
- A candidate two lenses both hit is ONE finding with both tags. Root cause is the
  dedupe key, not the symptom.
