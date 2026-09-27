---
name: memory-forge
description: >-
  Build and curate a durable, deduped agent memory store where every entry
  carries provenance, passes a live recall test before it counts as remembered,
  and no two entries are allowed to contradict (explicit supersede, never silent
  coexistence). Consumes what session-mine unearths, serves what recall loads,
  and stops when a curation pass changes nothing. Use when "make the agent
  remember this", "build a memory system", "the agent keeps forgetting X",
  "clean up / consolidate the memory files", memory that has grown stale or
  self-contradictory, or standing up cross-session memory for a new agent.
---

# Memory Forge

Most agent "memory" is a write-only diary: facts go in, nothing comes back out at
the moment it's needed, and after three months the store contradicts itself in
four places. This skill is the curation half of the memory loop: `session-mine`
unearths raw material, memory-forge smelts it into a store that actually fires,
and `recall` is the read path that serves it back at session start. The forge's
whole job is making three properties hold at all times: **every entry is traceable,
every entry is retrievable, and no two entries disagree.**

> **Core Insight:** A memory is not what you wrote down; it's what comes back when
> you need it. So the unit of acceptance here is not "the fact is in the file";
> it's a passed **recall test**: from a fresh context, given only a realistic
> trigger (the project dir, the person's name, the task shape), does the read path
> surface this entry, and does the entry then tell the agent what to DO? An entry
> that fails that test isn't memory, it's sediment, and sediment is worse than
> nothing, because it costs tokens on every load while returning zero behavior
> change.

> **Core Insight #2:** Contradiction is the store's rot. Two entries that disagree
> ("always use pnpm" / "this repo uses npm") don't average out. Whichever loads
> last wins at random, which means the store now injects noise with full
> confidence. The forge's law: **a conflict is resolved the moment it is seen,
> by supersession, never by deletion.** The losing entry stays, marked
> `superseded-by: <winner>` with the date and the reason. The history of what
> you used to believe is itself a memory, and it's how you notice a fact that
> keeps flip-flopping (which is a fact about the fact).

## The entry contract

No entry enters the store without all five:

1. **The memory itself:** one atomic claim. "Prefers X over Y for Z." Compound
   entries can't be superseded cleanly; split them.
2. **Trigger:** when this should surface (project, person, task shape, keyword).
   An entry with no trigger can never pass the recall test.
3. **Action:** what the agent should DO differently. "Note that…" is not an
   action; "use A, never B, when C" is.
4. **Provenance:** where this came from: the session (from `session-mine`, its
   ledger entry travels along), the user's words, the incident. Dated.
5. **Status:** `active` | `superseded-by: <id>`. Nothing else; there is no
   "probably still true".

## The firewall: curate the store, don't act on it

A forge pass writes MEMORY ONLY. Discovering, mid-pass, that a memory implies work
(a config is wrong, a skill should exist, a doc is stale) produces a routed
handoff, never an inline fix: methods that generalize → `skillify`; project work →
that project's backlog; new mining leads → `session-mine`. The pass that curates
and the pass that acts are different passes. A forge run that "also fixed a few
things" has an unreviewable diff and, worse, a memory store shaped by what was
convenient to fix.

## The pipeline (per curation pass)

1. **INTAKE:** gather candidates: session-mine dispositions, explicit "remember
   this" items, corrections from the live session. Normalize each against the
   entry contract; a candidate that can't name its trigger or action goes back.
2. **DEDUP:** search the store (keyword + trigger overlap) before writing.
   Duplicate → merge: keep one entry, union the provenance (two independent
   sources beat one). Near-duplicate with drift → that's a conflict; next step.
3. **RECONCILE:** every detected conflict resolved NOW: pick the winner (newer
   provenance wins by default; stronger evidence beats newer when they clash;
   a tested outcome outranks an offhand remark), mark the loser superseded with
   reason and date. A pass may not end with a known-unresolved conflict; if the
   winner genuinely can't be picked, the conflict escalates to the user as a
   decision item. It does not sit in the store as two active entries.
4. **RECALL-TEST** (the gate): for each entry written or changed this pass: from
   a fresh context, simulate the trigger (run the actual read path: `recall`
   with the project dir or query, or your store's search) and check both halves:
   it **surfaces** (retrievable) and it **applies** (the action is executable as
   written). Fail on surface → fix the trigger/placement. Fail on apply → rewrite
   the action. Re-test. An entry that can't pass after a rewrite gets demoted
   back to candidate; it is not "in memory".
5. **DECAY SWEEP** (periodic): sample the store's `active` entries against
   reality: still true? still triggered by anything that happens? An entry whose
   trigger hasn't fired in an age isn't deleted; it's challenged: verify it
   or supersede it. Dead weight fails recall tests eventually anyway; the sweep
   just finds it cheaper.

## Stopping condition: the converged pass

Run curation passes until **one full pass changes nothing**: no new entries
survive intake, dedup finds no merges, reconcile finds no conflicts, every
recall test that ran was already green. That pass is the proof of convergence
for the current corpus; new sessions re-open the forge, more polishing of the
same corpus does not. (A pass that only *reformats* entries does not count as
change; churn is not curation.)

## Composes with

- **`session-mine`:** upstream: its WORKED ledger entries arrive with provenance
  and outcome evidence already attached; its FAILED entries become anti-memories
  ("known-bad: this prompt/approach failed, see session …").
- **`recall`:** downstream: the read path, and the very machinery the recall
  test exercises. If recall's mappings can't route a trigger, that's a forge
  finding (fix the placement), not a recall bug to shrug at.
- **`skillify`:** the promotion path when a memory turns out to be a METHOD:
  a memory says *what's true*; a skill says *how to do it*. When an entry's
  "action" grows steps, it's outgrown the store.

## Anti-Patterns

- Write-and-forget: appending to memory files with no recall test; you're
  building sediment with good intentions.
- Deleting a wrong memory: you just erased the evidence that you once believed
  it, and with it the flip-flop signal. Supersede.
- Two active entries that disagree "because context differs." Then the context
  IS the trigger; encode it, or reconcile them.
- Vibes-based entries: no provenance ("I think we decided…"). If you can't say
  where it came from, it can't be trusted enough to auto-load.
- Compound mega-entries: un-supersedable; when half goes stale the whole entry
  lies.
- Fixing the world mid-pass: the firewall exists so the store isn't shaped by
  what was fun to fix.
- Optimizing for store size or entry count. The metric is recall-test pass rate,
  not volume; a 40-entry store that fires beats a 400-entry archive that doesn't.
- Endless polishing passes: one no-change pass is convergence; stop.

## THE EXACT PROMPT

```
/memory-forge
Store:      <memory root — e.g. ~/.claude/projects/<home-slug>/memory/, a vault dir>
Intake:     <what's arriving — session-mine ledger, "remember this" items, corrections>
Read path:  <how memories load — recall, a search tool, always-load files>
Mode:       <curate (default) | bootstrap (new store) | decay-sweep>
```
