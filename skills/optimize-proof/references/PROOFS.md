# Behavior Proofs & Measurement Protocol

Companion to optimize-proof. Two jobs: pin the target's observable behavior
*before* the change, and measure the change honestly *after* it.

## Part 1: Behavior proofs

A behavior proof is a test asserting the optimized code is observably identical
to the original. "Observably" = what callers and users can see: return values,
side effects, emitted records, error behavior, ordering where ordering is part
of the contract. Internal call counts, allocation patterns, and timing are NOT
behavior; they're exactly what the optimization is allowed to change.

Write the proof against the ORIGINAL code and run it green there first. A proof
that has never passed on the original proves nothing about equivalence.

### Recipe A: Golden proof (default)

Capture the original's outputs on representative inputs; assert the optimized
version reproduces them byte-for-byte (or field-for-field).

1. Pick 5–15 inputs from the *real workload*, not invented ones: the common case,
   the empty case, the largest case the workload contains, the weird unicode/
   null/duplicate-key cases the hot path actually sees.
2. Run the original; serialize outputs deterministically (sorted keys, stable
   float formatting, stripped timestamps/uuids/addresses) to
   `<target>/.perf/goldens/<hotspot-id>/`.
3. The proof test replays the inputs and diffs against the goldens.

Non-determinism in the ORIGINAL (map iteration order, parallel completion order)
must be normalized in serialization (sort before comparing) or the golden
flakes and the proof is dead weight. If the contract genuinely permits multiple
valid outputs, golden the *normalized* form and add a Recipe B invariant for the
rest.

### Recipe B: Invariant proof

When exact goldens are brittle (floats, legitimate output freedom), assert the
properties that constitute correctness instead: result length matches, every
input item is accounted for, totals balance, output is sorted by the documented
key, numeric results within an explicit epsilon (state the epsilon and why).

### Recipe C: Metamorphic / differential proof

The strongest option when you can afford it:

- **Differential:** keep the original implementation alive (renamed, or behind a
  flag) and run both on the same inputs in the test, asserting equality. Delete
  the original only after the change is banked. This is near-zero-thought
  equivalence checking; prefer it whenever the original is cheap to keep.
- **Metamorphic:** assert relations that must survive optimization:
  `f(shuffle(xs))` ≡ `f(xs)` for order-free contracts, `f(xs ++ ys)` consistent
  with `f(xs)` + `f(ys)` for mergeable ones, idempotence where documented.
  Catches the bugs goldens miss because no golden input happened to trigger them.

### Logging discipline (non-optional)

A proof that says `AssertionError` wastes the next hour. On divergence, the
proof must log: which input (by id/index), expected vs. actual (a real diff, not
two blobs), and the first point of divergence for sequences. Log the green path
too, one line per case (`case 7: 4,812 records, checksum ok`); silent-green
proofs hide tests that stopped exercising anything. Structured (JSON-lines) if
anything will parse it; greppable either way.

### Sizing the proof

Proportional to blast radius. A pure function hot loop: goldens + one
metamorphic relation, done in 20 minutes. A hotspot that touches persistence or
crosses a service boundary: differential proof + side-effect assertions (rows
written, messages emitted, files touched; compare counts AND content). If the
proof is becoming a bigger project than the optimization, that's signal the
hotspot's blast radius is large. Slow down, don't skip the proof.

## Part 2: Measurement protocol

1. **Real workload, end to end.** Measure the workload the user experiences (or
   the closest reproducible cut of it), not an isolated micro-benchmark of the
   changed function. Micro-benchmarks are allowed as a *diagnostic* during step
   (d), never as the banked number.
2. **Same conditions both sides.** Same machine, same data, same flags, caches
   in the same state (decide cold or warm and say which; warm = discard run 1).
   Quiet the box: no swarm panes, no model loads mid-run. On a unified-memory
   box (e.g. a DGX Spark/GB10 class machine), wrap heavy runs in
   `memguard run --profile perf -- CMD`: UMA means a hungry benchmark steals
   system RAM and a frozen box loses the numbers. Elsewhere, run bare.
3. **N runs, median + spread.** N≥5 for sub-second paths, N≥3 for multi-minute
   ones. Record median and min–max spread in OPTLOG, e.g.
   `before: 1.84s (1.79–1.97, n=5) · after: 0.41s (0.39–0.44, n=5) · 4.5×`.
4. **Noise floor rule.** The claimed delta must clear the spread. Median moved
   1.84→1.78 with a ±0.09 spread? That's noise: `reverted: no-win`. If the
   change "should" matter but doesn't clear the floor, the workload may be too
   small to show it: grow the workload first, then re-measure; don't lower the
   bar.
5. **Measure the right dimension.** The hotspot's score says what it costs.
   CPU-bound: wall + CPU time; memory: peak RSS (`/usr/bin/time -v`, or
   `/proc/meminfo` deltas on unified memory; never nvidia-smi on UMA); I/O: wall +
   bytes/ops; contention: wall under the *concurrent* workload (a lock fix
   measured single-threaded shows nothing).
6. **One change per measurement.** Stacked changes can't be attributed. When
   one of three regresses behavior, all three go back. Bank, then climb again.
