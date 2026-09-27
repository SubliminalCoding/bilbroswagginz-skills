# Complexity Metrics & The Drop Rule

Companion to simplify-proof. The proof recipes live in optimize-proof's
PROOFS.md (shared spine: golden / invariant / differential, the logging
discipline); this file covers the other half of the gate: measuring "simpler"
honestly.

## Part 1: The metric set (they travel together)

Measured per target, before step (d) and after every transform:

1. **Cyclomatic complexity:** sum over the target AND max per function. The sum
   catches sprawl; the max catches the god-function hiding behind a decent
   average.
2. **Max nesting depth:** the strongest proxy for "humans can't hold this";
   guard-clause and early-return transforms show up here first.
3. **LOC** (target total) and **longest function:** coarse, but honest across
   languages and immune to tool quirks.
4. Optional, when the tool offers it: **cognitive complexity** (Sonar-style),
   better aligned with readability than cyclomatic. Use it as a fifth column,
   not a replacement.

No single metric decides anything; that's the gaming guard (Part 3).

## Part 2: Tools per language (pick one, record which)

| Language | Tool |
|---|---|
| Python | radon cc / radon raw (or ruff's mccabe rule for max-per-function) |
| Rust | cargo clippy -W clippy::cognitive_complexity; tokei/cloc for LOC |
| JS/TS | eslint complexity + max-depth rules (report mode); cloc |
| Go | gocyclo + gocognit |
| Multi/fallback | lizard (cyclomatic + NLOC + function length, most languages) |

Same tool, same flags, before and after. A metric delta across two different
tools is noise wearing a suit. Record tool + version in SIMPLOG once per run.
No tool available for the language → count by hand for the target (branch points,
deepest indent, lines). The target is one module, not the repo; hand-counting
is minutes and keeps the gate honest.

## Part 3: What counts as a measured drop (the no-drop rule)

Banked requires: **at least one metric materially down, none materially up.**

- **Materially** = beyond formatting jitter: for cyclomatic and nesting, any
  whole-point drop on max or ≥10% on the sum; for LOC, ≥10% on the target.
  A 2-line drop on an 800-line module is inside the noise floor of a reformat:
  `reverted: no-drop`.
- **Trades are not drops.** Nesting −2 but cyclomatic sum +15% means complexity
  moved, it didn't leave. Sideways → revert. (One narrow exception: max-per-
  function down sharply while the sum holds. Splitting a god-function
  legitimately spreads the same branches across readable units. Log it as
  exactly that, and the *next* transform on the pieces should drop the sum.)
- **The gaming guards, named:**
  - *Code golf:* LOC down via density (ternary pileups, one-letter lambdas,
    clever chaining) while cyclomatic/nesting hold. Caught by co-measurement;
    also just don't.
  - *Complexity laundering:* extracting the ugly branch tree into a helper
    (or a config blob / lookup table of lambdas) and measuring only the now-
    pretty caller. Measure the TARGET = the module plus everything extracted
    from it this pass. Extraction relocates complexity; only deletion and
    collapse destroy it.
  - *Dead-code inflation:* deleting dead paths is a real and encouraged drop
    (transform #1), but verify dead means dead: no reference, no reflection/
    dynamic dispatch reaching it, no public export. "Probably dead" is an inbox
    item with a deprecation plan, not a deletion.

## Part 4: The performance tripwire (shared ground with optimize-proof)

Simplification usually costs nothing measurable. But on a KNOWN hotspot
(anything in a `.perf/HOTSPOTS.md` or OPTLOG), both siblings' gates apply: run
the hotspot's real workload before/after per optimize-proof's measurement
protocol (N runs, median + spread). A simplification that regresses a measured
hotspot beyond its noise floor is a trade the operator decides, not a default; inbox it
with both numbers. Off the hotspot list, don't measure performance at all;
inventing benchmarks for cold code is how simplification passes stall.

## Part 5: SIMPLOG entry format

```
## <target> — <date>
why-complex: <the one sentence>
transform:   <dead-path delete | dedup | flatten | extract/inline | rename-rider>
before:      cyc 84 (max 31) · nest 6 · loc 412 · longest 118
after:       cyc 41 (max 12) · nest 3 · loc 305 · longest 44
proof:       tests/characterize_<target>.py (green on original @ <sha>)
status:      banked | reverted: behavior | reverted: no-drop
inbox:       <warts pinned, API asks, perf trades — or none>
```
