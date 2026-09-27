---
name: project-autopilot
description: >-
  Autonomous project-pushing loop: extract the project's stated intention, diagnose
  gaps through relevant expert lenses, route each gap to the best installed skill,
  and execute in small green-verified batches until the intent is satisfied, queuing
  anything that needs your judgment into a decision inbox instead of guessing.
  Use when "push this project along", "autopilot this", "take this to done",
  "run off to the races", or unattended improvement of a project already in motion.
---

# Project Autopilot

Point it at a project that's already in motion (intent laid out, scaffold standing)
and it pushes the project toward done the way the operator would: one deliberate move at a
time, the right specialist skill for each move, green after every batch, and a tidy
stack of real decisions waiting at the end instead of guesses made along the way.

> **Core Insight:** The Swiss Army knife is the routing table, not one giant blade.
> Autonomy comes from three things working together: a stated intention to measure
> against, a green ratchet so the loop can never make things worse, and a decision
> inbox so it never makes judgment calls on the operator's behalf. AI handles everything that's
> engineering; everything that's judgment queues for the human.

> **Core Insight #2: this is a successive-prompter, not a megaprompt.** The skill's
> real value is *timing*, not just routing. The operator's most effective workflows are
> staged prompts delivered one at a time: orient, then run the big skill, then
> review it, then make beads, then polish, then implement, never all at once. The
> later instructions are withheld on purpose: hand a model the whole pipeline up
> front and it tries to hold everything at once and does each step worse; give it
> one timed instruction at a time and it focuses and does each step well. **This
> skill plays the role the operator plays when prompting at each level: it pushes the work
> along one move at a time, delivering the right instruction only when the prior
> step completes.** Do NOT dump all phases into one prompt. The tick loop IS the
> mechanism: one move per tick, the next instruction issued only after the last
> verified green.

## Preconditions (check before tick 1; refuse politely if missing)

1. **Intent + a scaffold exist.** A README, plan doc, or notes file states what the
   project is trying to be, and there's a working scaffold to push on. No intent =
   run reality-check's intent extraction FIRST and confirm with the operator.
   **True greenfield (a bare idea, no scaffold yet) = wrong tool: run /kickstart
   first** to reach a walking, intent-aligned scaffold, then autopilot takes the baton.
2. **Git repo with a reachable baseline.** The ratchet needs checkpoints.
   The tick loop also needs `br` (beads, the pack's one required dependency;
   see DEPENDENCIES.md at the pack root).
3. **Baseline is green** (or the operator explicitly accepts the current red as baseline;
   see SAFETY-RAILS escalations).
4. **Late-stage guard:** >10 modified files or >3 in-flight beads at launch →
   stabilize what's in flight before any new diagnosis (standing rule).

## Phase 0: Launch

```
1. Read intent sources (README, plan docs, any project notes in the repo)
   → write Intent Statement to <project>/.autopilot/INTENT.md (one paragraph:
   who it serves, what job it does, what "winning" looks like, commercial vs fun)
2. Load prior context: `recall` (project-aware memory) + `session-mine` / `memory-forge` (prior-session patterns)
   for this project (prior sessions, decisions, known traps)
2b. SHIPPED-REALITY PROBE (per LENSES.md): git remote? deployed where the user
   can reach it? anyone but the operator ever used it? does the headline promise have an
   UNMOCKED e2e proof? Record findings: a goal proven only by mocks is STUB at
   shipped level. Loop-closing gaps (repo/deploy/domain) are usually ask-user and
   belong at the TOP of the inbox; they outrank localhost polish.
3. Relevance-filter lenses per references/LENSES.md → record the chosen set
4. Baseline: build + tests + smoke → checkpoint commit
5. Mode select: solo (this session loops) | swarm (parallel agents execute under
   `swarm-run`'s two contracts, sole-writer ownership map + orchestrator-re-run
   gate; autopilot plays operator per references/OPERATOR-MOVES.md cadence).
   Swarm mode is NOT assumed; it's earned by a capability check at IMPLEMENT time.
   See references/EXECUTION.md "Execution backend: detect & degrade". Solo-sequential
   is always the floor; a missing Codex sub or ntm NEVER blocks the loop.
```

## The Tick Loop

```
ASSESS    What's true right now? tests green · br ready count · open inbox items
          · gaps remaining vs INTENT.md. If bead graph is empty → run a full
          lens diagnosis (LENSES.md) and convert gaps to beads (beadsmith).
DECIDE    Pick ONE next move. Consult references/OPERATOR-MOVES.md ("what would
          the operator do at this state?") + cm trauma guard (if installed). Priority = impact_on_intent
          ÷ effort, with correctness > everything when red anywhere.
ROUTE     Map the move to a skill via references/SKILL-ROUTING-MAP.md.
          mode=ask-user → write to .autopilot/DECISIONS.md, pick next move instead.
EXECUTE   Invoke the routed skill on a batch of ≤5 beads / ≤10 files.
VERIFY    Build + tests + smoke. Red after 2 fix attempts → revert to checkpoint,
          tag the gap `failed-auto` into the inbox, move on.
LOG       Checkpoint commit · br close/update · cm learning if something novel
          · append one line to the project's run log (e.g. .autopilot/LOG.md)
CONVERGE? Check references/CONVERGENCE.md (bead drain → 2 dry passes → final
          reality-check). Done or budget-hit → Phase 6. Else → tick again.
```

Tick cadence: solo mode just continues in-session. For unattended runs, drive
with /loop or an ntm looper (if installed); the loop is resumable from beads + inbox +
checkpoints alone (see CONVERGENCE.md Re-entry), so interruption is free.

## Heavy skills & execution backend → references/EXECUTION.md

Two EXECUTE-step concerns live in [EXECUTION.md](references/EXECUTION.md), pulled
out to keep this entry point lean:

- **Big-Skill Pipeline:** when the move is a heavy analysis/audit (security audit,
  codebase audit, bug hunt, optimization sweep), stage it: orient → apply
  exhaustively → **smart-model self-review** (non-skippable: the top model re-checks
  the subagents' findings) → beads → polish ×3 → implement. Never pipe a big skill's
  raw output straight to beads.
- **Execution backend (detect & degrade):** the swarm is an accelerator, never a
  precondition. Probe at implement time and pick the best live rung: ntm+Codex →
  ntm+CC → other-model / another runtime's delegate-task → in-process Agent/Workflow → **solo CC
  sequential (the floor)**. A lapsed Codex sub NEVER blocks the loop.

## Phase 6: Done Report

Write `<project>/.autopilot/REPORT.md`:

```
# Autopilot run: <project>, <date>
Status: done | budget-exhausted | stalled | unstable
Intent: <the statement it measured against>
Shipped: <per-tick one-liners with commit SHAs>
Decision inbox: <N open items: the things that need YOU, recommended option first>
Declined polish: <low-impact gaps deliberately left>
New-skill candidates: <gaps no installed skill covered>
```

Then surface the inbox items to the operator directly. The report is the artifact, the
decisions are the conversation.

## THE EXACT PROMPT

```
/project-autopilot

Target: <project path>
Mode: solo (default) | swarm (optional; needs ntm installed)
Budget: <max ticks, default 25>
Intent source: <README | plan doc | notes file path, if not obvious>
```

## Anti-Patterns

- Diagnosing without an Intent Statement (optimizing toward "generic best practice")
- Answering your own decision-inbox items because the answer "seems obvious"
- Mass-dispatch: many skills/agents mutating at once in solo mode
- Widening scope mid-run ("while I'm here…"); new ideas are inbox items
- Skipping the smoke check because tests pass (tests lie; running apps don't)
- Grinding polish beads while the headline promise has never run for a real user
  (the shipped-reality probe exists to catch this; close the loop first)
- Re-diagnosing from scratch on resume when beads + inbox already hold the state
- Treating budget-exhausted as failure: it's a checkpoint; relaunch continues

## References

- [LENSES.md](references/LENSES.md): diagnostic facets + relevance filter
- [SKILL-ROUTING-MAP.md](references/SKILL-ROUTING-MAP.md): gap → skill, auto vs ask-user
- [OPERATOR-MOVES.md](references/OPERATOR-MOVES.md): the mined operator steering moves, the DECIDE step's brain
- [EXECUTION.md](references/EXECUTION.md): big-skill pipeline + execution-backend ladder
- [CONVERGENCE.md](references/CONVERGENCE.md): green ratchet, done criteria, budgets
- [SAFETY-RAILS.md](references/SAFETY-RAILS.md): batch limits, memory guards, forbidden actions, escalations
