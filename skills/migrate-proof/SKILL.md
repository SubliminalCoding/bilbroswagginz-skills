---
name: migrate-proof
description: >-
  Carry a schema / data / framework / service migration from old to new under a
  parity gate and a rehearsed rollback: strict expand → migrate → contract, where
  contract is FORBIDDEN until a parity oracle shows old and new produce identical
  results on real traffic AND the rollback has been executed once in test. Use when
  "migrate the schema", "move to the new service", "swap the framework/library",
  "cut over to", "backfill and switch", or any change where old and new must
  coexist before the old is allowed to die.
---

# Migrate Proof

The lifecycle skill for the scariest class of change: the one where something that
works gets replaced by something that should. It's stack-general on purpose. A
column rename, a message-format change, a datastore swap, a framework port, a
service replacement all fail the same two ways at the same two ends. The shape is
always **expand → migrate → contract**, and the whole skill is the discipline of
refusing to contract early.

> **Core Insight:** Migrations don't fail in the middle; they fail at the ends:
> cutting over before proving the new path produces what the old one did, and
> contracting (deleting the old path) before proving you can come back. So the gate
> is double and non-negotiable: **contract is forbidden until (a) a parity oracle
> has shown old and new identical on real traffic, and (b) the rollback has been
> executed once, in test, recently.** A rollback plan that has never run is a wish
> with a runbook attached.

> **Core Insight #2:** Expand and contract are asymmetric. Every expand step is
> cheap and reversible: an unused column, a shadow deploy, a dual-write behind a
> flag cost nothing but disk and patience. Every contract step is where the point of
> no return lives. So spend all the courage early and all the caution late: expand
> aggressively, soak boringly, contract in the smallest reversible slices, each one
> a button. When unsure which side a step is on, it's a button (ship-it discipline).

## Honest overlap

The arsenal may have a database-migration how-to (if installed) for engine-specific
mechanics (locks, online DDL, index builds). This skill is the stack-general **gate**
that wraps any migration; when the migration is a database, run both: that one
drives the DDL, this one decides when contract is allowed to happen.

## Preconditions / guards

1. **The old system is green and observable.** You can't prove parity against a
   baseline you can't measure. Establish the old path's outputs/metrics first.
2. **Real traffic is reachable safely:** a sandbox/staging that sees real traffic,
   a replay of captured production requests, or prod shadow-reads. Synthetic-only
   parity is a guess wearing a lab coat.
3. **Checkpoint + the irreversible list written up front.** Phase 0 enumerates every
   step that can't be undone (dropped columns, deleted topics, DNS cutover, purged
   backups); each is pre-tagged as a button. No irreversible step may be discovered
   mid-flight.
4. **The green ratchet holds throughout.** Both paths' tests stay green at every
   phase; a migration step that reds either path reverts.

## The Pipeline (staged: each phase's exit gate holds before the next begins)

```
Phase 0  MAP        Inventory EVERY reader and writer of the old shape (grep is not
                    optional; the one cron job you missed is the outage). Write
                    <target>/.migrate/PLAN.md: expand steps, the parity oracle, the
                    rollback per phase, contract steps, each tagged
                    reversible|BUTTON. Checkpoint.
Phase 1  EXPAND     Build the new alongside the old: nullable column, dual-write
                    behind a flag, shadow-deployed service, the new framework path
                    mounted dark. The old path stays untouched and AUTHORITATIVE.
                    Exit: new path exists, receives real writes/requests, and
                    nothing user-visible changed.
Phase 2  MIGRATE    Backfill history; run the parity oracle on real (sandboxed)
                    traffic (recipes in references/PARITY.md). Every divergence is
                    explained and fixed (in the new path, or as a documented
                    accepted difference the operator signs off). Exit: parity boringly
                    green over the agreed soak window.
Phase 3  REHEARSE   Execute the rollback ONCE, in test: flip authority back to old,
                    prove the system runs on it, prove no data written during the
                    new-authority window was lost (references/PARITY.md Part 3).
                    Exit: a rollback that has actually run, timed, with its command
                    sequence written down verbatim.
Phase 4  CUTOVER    Flip authority to new. Production cutover is a BUTTON (the operator's),
                    with the rehearsed rollback named beside it. Old path keeps
                    receiving writes (or capturing them) so the rollback stays live.
                    Re-run the oracle post-cutover; watch it, don't assume it.
Phase 5  CONTRACT   FORBIDDEN until: oracle green through the post-cutover soak +
                    rollback rehearsed + the operator's explicit go. Then contract in
                    slices, cheapest-to-undo first: stop dual-write → remove old
                    code path → archive old data → drop, each slice its own
                    button with its own (shrinking) rollback. Exit: old path gone,
                    PLAN.md closed out as the audit trail.
```

## Contract with the other skills

- **ship-it** owns the button discipline this skill borrows: cutover and every
  contract slice are surfaced exactly like publish buttons: the command, the blast
  radius, the rollback, the operator's go per action.
- **test-max** hardens the parity oracle's assertions when the migration guards a
  money/data path (a theater oracle is worse than none: it green-lights the loss).
- **trace-it** takes over when the oracle reports a divergence nobody can explain:
  instrument until the trace shows where old and new part ways, then fix no-slop.
- **project-autopilot / forge** route "replace X with Y" gaps here rather than
  letting a tick loop attempt a big-bang swap.

## THE EXACT PROMPT

```
/migrate-proof
Target:   <project path>
Migration: <old → new, in one line: schema/data/framework/service>
Traffic:  <the real traffic source for parity — sandbox, replay capture, shadow>
Soak:     <parity window before cutover / before contract; default 48h or 10k
           requests, whichever is later>
```

## Anti-Patterns

- Contracting before the parity oracle and the rollback rehearsal both hold: the
  cardinal sin. Everything else in this file exists to prevent it.
- Big-bang cutover with no expand phase ("it's a small change"). Small changes
  with dual paths cost a flag; small changes without them cost a weekend.
- Migrate and contract in the same change/PR. When it breaks, the old path you
  needed is in the same revert as the fix.
- A rollback "plan" that has never been executed. Phase 3 exists because the
  rehearsal always finds the missing grant, the stale credential, the data written
  meanwhile.
- Parity by row counts alone. Counts match while every value is wrong; the oracle
  compares content (checksums + sampled field-level diffs).
- Proving parity on synthetic traffic only. Real traffic carries the unicode, the
  nulls, the legacy rows that synthetic never does.
- Letting the dual-write flag rot for months. Contract is the finish line; gate
  it hard, but get there. Two paths forever is a migration that failed slowly.
- Discovering an irreversible step mid-flight. The Phase 0 list was incomplete;
  stop, re-map, re-plan before touching anything else.

## References

- [PARITY.md](references/PARITY.md): parity-oracle recipes, divergence triage,
  and the rollback rehearsal protocol
