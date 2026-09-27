# Parity Oracles & Rollback Rehearsal

Companion to migrate-proof. Two jobs: prove old and new produce identical results
on real traffic, and prove the way back actually works.

## Part 1: Parity oracle recipes

A parity oracle is an automated comparison of old-path and new-path results on the
same real inputs, run continuously through the soak. Pick the strongest recipe the
migration affords; combine when they're cheap.

### Recipe A: Dual-read compare (default for data/schema)

Both paths live (expand phase makes this free): serve from old, compute from new,
compare before returning, emit a structured divergence record on mismatch
(request id, field, old value, new value). Sample if compute is expensive, but
sample real traffic, never a curated subset.

### Recipe B: Shadow traffic (default for service/framework swaps)

Mirror real requests to the new path; diff full responses (status, headers that
matter, body) against the old path's actual responses. Normalize legitimate
nondeterminism before diffing (timestamps, generated ids, ordering the contract
doesn't promise), the same normalization discipline as optimize-proof's golden
proofs. An oracle that flakes on noise gets muted by humans within a week and is
then worse than no oracle.

### Recipe C: Batch reconciliation (default for backfills)

Over the full migrated dataset: row/entity counts per partition, checksums per
partition (order-normalized), plus field-level diff on a random sample big enough
to bound the error you care about (not 10 hand-picked rows). Re-run on a schedule
during dual-write. Reconciliation that passed once and drifted since proves
nothing about cutover day.

### Divergence triage (every mismatch lands in exactly one bucket)

1. **New path bug** → fix in the new path (no-slop: THE divergence, not a refactor),
   oracle must re-run green over the affected slice.
2. **Old path bug faithfully NOT reproduced:** the new path is "right", the old is
   wrong. This is a behavior change hiding inside a migration: decision-inbox it
   for the operator. Either the new path reproduces the wart for now (bug-for-bug parity,
   fix after contract) or the operator signs the intentional change. Never silently ship it.
3. **Accepted difference** (documented, signed) → add to the oracle's normalization
   with a comment naming the sign-off. The accepted-differences list is part of
   PLAN.md.

Soak exit: zero unexplained divergences over the agreed window on real traffic.
"Boringly green": a soak that never fired once should make you check the oracle
is actually comparing (plant one deliberate divergence and watch it alarm; an
oracle that can't fail is theater, and test-max logic applies to oracles too).

## Part 2: What "real traffic" means

In descending order of strength: prod shadow (read-only mirror) → captured prod
replay into a sandbox (scrub secrets/PII per policy, keep the weird payloads;
they're the point) → staging with organic usage. Synthetic generators are allowed
only as a *supplement* for load, never as the parity basis. The rows that kill
migrations are the 2014 rows with the encoding nobody remembers.

## Part 3: Rollback rehearsal protocol (Phase 3)

Execute, don't review. In the sandbox, with the migration in its post-cutover
state (new path authoritative):

1. **Write through the new path first:** N real-shaped writes while new is
   authoritative. These are the canaries; rollback that loses them fails.
2. **Execute the documented rollback:** the exact command sequence from PLAN.md,
   run as written, timed. Any improvisation needed = the runbook was wrong; fix
   the runbook and re-run from step 1.
3. **Prove the system runs on old:** suite green + smoke on the rolled-back path.
4. **Prove the canaries survived:** the writes from step 1 are present and correct
   on the old path (this is what dual-write/capture during cutover is FOR; if they
   can't survive, that data-loss window is a named, sized, operator-signed risk on the
   cutover button, not a surprise).
5. **Record:** wall-clock time to rolled-back-and-green, the verbatim sequence, the
   canary result. This record sits next to the cutover button.

Re-rehearse if the migration's shape changes materially after Phase 3. A
rehearsal of last month's plan blesses last month's migration.

## Part 4: Contract slicing (Phase 5 order)

Cheapest-to-undo first, one button each: (1) stop dual-write (undo: restart it +
re-backfill the gap) → (2) remove old code path (undo: git revert) → (3) archive
old data to cold storage (undo: restore) → (4) drop/delete (undo: none; this is
the last button for a reason). After each slice, the oracle (or its post-contract
remnant: error rates, reconciliation of what remains) runs once more before the
next slice.
