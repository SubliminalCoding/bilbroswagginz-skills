---
name: idea-forge
description: >-
  Ideation engine: ground in the project's reality, diverge to ~30 candidate
  improvements, winnow ruthlessly to a ranked shortlist, then operationalize the
  winners into a self-documenting br bead graph, refined over multiple plan-space
  passes before a line of code is written. Use when "what should we build next",
  "brainstorm improvements", "ideation pass", "fill the backlog", "turn ideas into
  beads", or when forge/kickstart/project-autopilot need their ideation stage.
---

# Idea Forge

Point it at a project and it does what a great staff engineer does in a planning
week: learns what the project actually is, generates far more ideas than it keeps,
kills most of them with reasons, and turns the survivors into a dependency-ordered
task graph so complete that the planning doc becomes unnecessary. All of it happens
in plan-space (cheap to revise, free to throw away) before implementation spends
a single token on code.

> **Core Insight:** Idea quality is bought with *volume then violence*: generate
> ~30, keep ~15, and make the winnow do real reasoning per candidate, not vibes.
> But ideas are worthless until they're operationalized: the output is not a list,
> it's a **bead graph where every task body is self-contained** (background,
> rationale, success criteria, tests). The graph replaces the plan doc. And because
> a first-draft graph is always wrong somewhere, the last phase is 4–5 refinement
> passes over the *beads*, the cheapest possible place to fix a bad plan.

> **Core Insight #2:** Ideation is top-model work, full stop. The diverge, winnow,
> and review phases run on Opus/Fable-class reasoning, never a local model. A weak
> model generates plausible-sounding ideas that are subtly wrong for the project;
> you can't winnow your way out of a bad candidate pool. (There is deliberately no
> qwen- variant of this skill.)

## Preconditions

1. **A real project:** AGENTS.md / README.md / CLAUDE.md or equivalent intent
   source exists. No intent = run reality-check's extraction first.
2. **br initialized** (`br list` works in the project). The output IS beads; no
   tracker, no skill. (beads is the pack's one required dependency; see
   DEPENDENCIES.md at the pack root.)
3. **Late-stage guard:** >10 modified files or >3 in-flight beads at launch →
   ask the operator before running (standing rule: mass new beads mid-iteration
   destabilize what's working).

## The Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  GROUND      Read AGENTS.md / README.md / CLAUDE.md + skim the code's
                     shape. Pull the EXISTING task graph, open AND closed:
                     `br list --status=open` and `--status=closed`. Closed beads
                     show what's already been done; open ones what's already
                     planned. Ideas must fit this reality, not a generic project.
Phase 1  DIVERGE     Generate ~30 candidate improvements spanning the rubric
                     dimensions (references/RUBRIC.md): robust, reliable,
                     performant, intuitive, user-friendly, ergonomic, useful,
                     compelling. Constraint: obviously ACCRETIVE and PRAGMATIC:
                     no moonshots, no rewrites, no scope explosions. Number them.
Phase 2  CONVERGE    For EACH candidate, reason it through: how it works, how a
                     user perceives it, how it's implemented here, complexity vs
                     payoff. Then winnow to the best ~5, ranked best→worst, each
                     with full rationale (why it won AND what it cost the losers).
Phase 3  GATE 1      Smart-model review (NON-SKIPPABLE). Fresh eyes on the winnow:
                     re-check the top 5 against the actual codebase: is each one
                     real, novel vs the existing graph, and ranked right? Demote /
                     swap candidates that don't survive contact with the code.
Phase 4  EXPAND      Pull the next ~10 (ranks 6–15). They're often the
                     complementary angles the top 5 miss: the logging to the top
                     5's features, the docs to their APIs. → ~15 total winners.
Phase 5  DEDUP       Cross-check all 15 against open beads. Overlap → MERGE into
                     the existing bead (enrich its body), never duplicate.
Phase 6  TRIAGE      Decision-inbox split: any winner that is irreversible, costs
                     money, or sets product direction → write to
                     <project>/.autopilot/DECISIONS.md (recommended option first)
                     and EXCLUDE from auto-operationalization. The operator decides; the
                     skill doesn't.
Phase 7  OPERATIONALIZE  Remaining winners → granular br graph: epic → task →
                     subtask, dependency-ordered. EVERY body self-contained
                     (template below). Explicit test task + logging task per epic.
                     Then `bv` to validate: dependency graph must have NO cycles.
Phase 8  GATE 2      Smart-model review of the DRAFT graph (NON-SKIPPABLE): does
                     each bead make sense standalone? Right granularity? Deps
                     real? Anything oversimplified or silently dropped?
Phase 9  REFINE      4–5 passes over the graph in plan-space. Each pass:
                     re-ground in AGENTS.md, walk every bead, ask "is this still
                     sensible and optimal?" Hard constraints every pass:
                     DO NOT OVERSIMPLIFY · DO NOT DROP FEATURES/FUNCTIONALITY ·
                     tests with detailed logging stay in. Stop at 5 passes or
                     when a pass changes nothing.
```

## The self-contained bead body (Phase 7 template)

Each bead must read like a brief a contractor could execute with zero context:

```
WHAT      one-sentence deliverable
WHY       background + rationale — what problem, why now, which winner it serves
HOW       approach + key considerations / gotchas specific to THIS codebase
DEPENDS   what must land first (mirrored as real br deps)
DONE WHEN observable success criteria — including the test that proves it and
          what its logging must show
```

`br create` the epic, `br create` tasks with `--deps` (or `br dep add`), label
everything `idea-forge` so a later run can tell its own output from hand-written
beads. `bv` for triage order + cycle check before declaring Phase 7 done.

## Contract (what orchestrators can rely on)

forge, kickstart, and project-autopilot consume this skill as their ideation
stage. The interface is fixed:

- `<project>/.ideaforge/IDEAS.md`: full numbered candidate log + winnow rationale
  (the audit trail; safe to delete after the run).
- `<project>/.ideaforge/WINNERS.md`: the ranked ~15 with one-line rationale each
  and the bead ID each became (or `inbox` / `merged:<id>`).
- **br:** the operational output. All created beads labeled `idea-forge`.
- `<project>/.autopilot/DECISIONS.md`: appended judgment items (same inbox
  project-autopilot drains; one inbox, not two).
- Exit state: `bv` clean (no cycles), refinement passes complete. Nothing
  implemented; idea-forge ends where execution begins.

## THE EXACT PROMPT

```
/idea-forge
Target: <project path>
Focus:  <optional — a dimension or area, e.g. "reliability" | "onboarding UX">
Count:  <winners to operationalize, default 15>
```

## Anti-Patterns

- Ideating before grounding: generic "best practice" ideas that don't fit the repo.
- Skipping closed beads in Phase 0. You'll proudly propose what shipped last month.
- Vibes-winnowing. Every kill and every rank needs the four-question reasoning.
- Skipping either review gate because the draft "looks good." The gates exist
  because first drafts always look good to their author.
- Creating a duplicate bead instead of merging into the open one.
- Auto-operationalizing a product-direction call. That's an inbox item, full stop.
- Beads whose body says "see plan doc." The graph REPLACES the doc; self-contained
  or it doesn't ship.
- Refinement passes that "simplify" by dropping features or deleting test/logging
  tasks. Refinement tightens wording and deps, never scope.
- Starting implementation. Hand off to project-autopilot / the swarm; plan-space ends here.

## References

- [RUBRIC.md](references/RUBRIC.md): the ten dimensions expanded, the four
  convergence questions, and scoring guidance for the winnow
