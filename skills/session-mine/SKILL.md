---
name: session-mine
description: >-
  Mine past agent/coding sessions for reusable prompts, decisions, and patterns,
  with the receipts: every extracted pattern is traced to the session where it ran
  AND its recorded outcome, so you never cargo-cult a prompt that actually failed.
  Dedups against what's already captured, stops when fresh sessions stop yielding
  novel patterns, and hands keepers to memory-forge / skillify instead of adopting
  them mid-mine. Use when "what did I ask last time", "find that prompt that
  worked", "what did we decide about X", "mine my history", session archaeology,
  or before re-deriving an approach you suspect a past session already nailed.
---

# Session Mine

Your session history is a lab notebook nobody reads: hundreds of experiments, each
with a prompt, a context, and (crucially) a result. This skill reads it. Point it
at your session logs (Claude Code JSONL, Codex history, a local agent's
logs, whatever the harness writes) and it extracts the prompts, decisions, and recurring moves worth
keeping, as *evidence-backed claims*, not nostalgia. It is the finder half of the
memory loop: `memory-forge` curates what this skill unearths, `recall` serves it
back at session start, `skillify` promotes the methods that generalize.

> **Core Insight:** Frequency is a lead, not proof. The prompt you typed ten times
> might be your best ritual, or a habit that failed ten different ways, which is
> why you kept re-typing it. The only honest test of a "working prompt" is what
> happened NEXT in the session where it ran: did the agent produce the thing, did
> the tests go green, did you move on, or did the next user message start with
> "no, that's not what I meant"? So the gate here is **provenance + outcome**:
> no pattern leaves the mine without the session it worked in and the evidence
> that it worked. A pattern without an outcome is an anecdote with a filename.

## The firewall: mine, don't adopt

This skill EXTRACTS. It never installs. No editing CLAUDE.md, no rewriting a skill,
no "while I'm here" adoption of a freshly-mined prompt into the current task:
extraction and adoption are different activities with different reviewers. Every
keeper gets a **disposition**, and the disposition is a handoff:

- durable fact / preference / decision → `memory-forge` (which owns dedup against
  the store and the recall test)
- a METHOD that generalizes across projects → `skillify`
- a project-specific lesson → that project's CLAUDE.md, **proposed**, not written
- everything else → dropped, with the reason logged

The mine's output is a ledger, not a changed system.

## Preconditions / guards

1. **A mining question.** "Mine everything" produces a museum. Every run starts
   from a question ("how did I get X working", "what prompts start my best
   sessions", "when did we decide against Y") that decides what a hit looks like.
2. **Read-only over the corpus.** Session logs are evidence; never edit, reformat,
   or "clean up" a session file. Chain of custody matters when a pattern's worth
   is its provenance.
3. **The existing capture set is loaded first**: memory store, existing skills,
   CLAUDE.md lessons. You can't dedup against what you haven't read, and the most
   common mining failure is triumphantly re-discovering something already captured.

## The pass structure

**Per pass (one mining question):**

1. **SWEEP**: locate candidate sessions. Keyword search across the corpus, plus
   the high-yield strata (sessions just before a ship, sessions that ended in a
   commit, sessions you returned to after context loss; what you searched for
   after losing context is what mattered). Rank by likely yield, take the top N.
2. **EXTRACT**: from each session, pull candidate patterns: the exact prompt or
   decision text, verbatim, with file + position + date. A paraphrase is not a
   candidate; the whole point is the wording that worked.
3. **TRACE THE OUTCOME** (the gate): for each candidate, read *forward* in the
   session until the outcome is visible, and record it as evidence: the artifact
   produced, the green run, the user's next message accepting it, or the
   correction, the retry, the abandonment. Three verdicts:
   - **WORKED**: outcome evidence in hand → eligible keeper.
   - **FAILED**: the session shows it didn't land → recorded in the ledger AS a
     failure (a known-bad prompt is real intel; it inoculates future mining), never
     dispositioned as a keeper.
   - **UNRESOLVED**: session ends before the outcome → stays a candidate; it may
     resolve in a sibling session (work clusters), one look, then drop or park.
4. **DEDUP**: against the capture set and this run's ledger. Same pattern, new
   session → don't re-record; *increment its evidence* (three independent WORKED
   traces beat one). Contradicting evidence (worked in April, failed in June) →
   record both; the conflict is the finding, and `memory-forge` owns resolving it.
5. **DISPOSITION**: route each keeper per the firewall. Log the routing.

**Ledger entry (all five fields or it doesn't enter):** pattern (verbatim) ·
provenance (session file/position, date, project) · outcome evidence (one
sentence + where to see it) · verdict · disposition.

## Stopping condition: saturation

Stop when **two consecutive sweeps yield nothing novel**: every extraction dedups
against the capture set or the run's own ledger. That's the corpus telling you this
question is mined out; the next marginal session costs more to read than it yields.
Then stop *for that question*. A new question re-opens the mine; more history
never does. If saturation arrives suspiciously fast, the sweep is too narrow
(one keyword, one harness, one project): widen once, then trust the signal.

## Anti-Patterns

- Extracting a prompt because it *reads* well. Eloquence is not evidence; trace
  the outcome or it's not a keeper.
- Counting repetition as validation. Ten retries of a failing prompt is the
  strongest possible signal, in the other direction.
- Adopting mid-mine: pasting a freshly-mined prompt into the live task, editing
  CLAUDE.md "since I found this". Disposition, then act in a separate step.
- Paraphrase capture: recording your summary of the prompt instead of the prompt.
  The wording IS the pattern.
- Mining without loading the capture set. You'll re-discover your own store and
  call it yield.
- Dropping FAILED traces because they're not keepers. The failure ledger is half
  the value; it's what stops the next agent from cargo-culting.
- Grinding past saturation because the corpus is big. Size isn't yield; two dry
  sweeps is the exit.
- "Cleaning up" session logs while mining. Evidence is read-only.

## THE EXACT PROMPT

```
/session-mine
Corpus:   <where the session logs live: harness dirs, log paths>
Question: <what you're mining for: a prompt, a decision, a recurring move>
Capture:  <the existing capture set to dedup against: memory store, skills, CLAUDE.md>
Budget:   <optional: max sessions per sweep; default 20>
```
