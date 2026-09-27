---
name: beadsmith
description: >-
  Plan-to-graph converter: take an EXISTING agreed plan (a plan doc,
  dueling-idea-forge consensus, hand-written markdown) and forge it into a clean,
  dependency-ordered br bead graph: every bead self-contained, every edge real,
  bv-clean, validated and polished in plan-space before any code is written. Use
  when "turn this plan into beads", "plan to beads", "beadify this", "create the
  task graph from the plan", "convert the markdown plan", or when kickstart /
  project-autopilot / ux-maximizer need their plan→tracked-work stage.
---

# Beadsmith

The bridge between planning and implementation. A plan doc is written for humans;
a swarm executes a graph. beadsmith does the translation a great tech lead does
before sprint one: decompose the plan into right-sized work items, wire the real
dependency edges, make every item executable by a contractor with zero context,
then audit the graph against the plan until nothing was lost in translation. It
does NOT ideate: the plan is already agreed. It does NOT implement: it ends
where the swarm begins.

> **Core Insight:** The failure mode of plan→tasks conversion is *silent scope
> loss*: the doc says ten things, the tracker holds seven, and nobody notices
> until the "done" project is missing features. So beadsmith is coverage-driven:
> every commitment in the plan must trace to a bead, the trace is written down,
> and the audit happens in plan-space where a fix costs an edit, not a sprint.
> The graph isn't a summary of the plan. It REPLACES the plan.

## Boundary vs idea-forge

idea-forge GENERATES ideas and operationalizes its own winners. beadsmith is the
standalone converter for a plan that already exists and is already decided. No
diverge, no winnow, no ranking. If you're holding a vague wish instead of an
agreed plan, run idea-forge or planning-workflow (if installed; otherwise write
the plan by hand) first. (Future refactor: the bead-writing here and idea-forge's
Phase 7 are the same craft; idea-forge could call beadsmith for the actual
bead-writing. Noted, not done; don't modify idea-forge.)

## Preconditions

1. **An agreed plan artifact:** a planning-workflow doc, a dueling-idea-forge
   consensus, decision-inbox resolutions, or a hand-written markdown plan. A
   one-line idea is not a plan; bounce to planning-workflow (if installed) or
   ask for a written plan.
2. **br initialized** (`br list` works in the project). The output IS beads.
   (beads is the pack's one required dependency; see DEPENDENCIES.md at the
   pack root.)
3. **Late-stage guard:** >10 modified files or >3 in-flight beads at launch →
   ask the operator before flooding the tracker (standing rule: mass new beads
   mid-iteration destabilize what's working).

## The Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  INGEST      Read the plan end-to-end + the project's intent sources
                     (AGENTS.md / README.md / CLAUDE.md). Inventory the EXISTING
                     graph (`br list --status=open` AND `--status=closed`) so
                     you merge into open beads instead of duplicating, and don't
                     re-plan what already shipped.
Phase 1  EXTRACT     Walk the plan and list every COMMITMENT it makes: each
                     feature, behavior, constraint, migration, doc, and "we
                     will" sentence. Number them. This list is the coverage
                     contract for the whole run; nothing on it may vanish.
Phase 2  SHAPE       Decompose commitments into epic → task → subtask. Sizing
                     test: one task = one focused agent session ending in a
                     commit that keeps the build green. Bigger → split; an
                     epic of one task → flatten. Add an explicit TEST task and
                     a LOGGING/observability task per epic; they are scope,
                     not garnish.
Phase 3  WIRE        Dependency edges between nodes: only REAL ordering ("B
                     touches what A creates"), never vibes-ordering ("A feels
                     first"). Walk the draft in topological order once: if you
                     can't, you've drawn a cycle. Fix it now, in outline form.
Phase 4  FORGE       `br create` epics, then tasks with `--deps` (or
                     `br dep add` after). EVERY body self-contained (template
                     below). Label everything `beadsmith` plus a per-run plan
                     slug so a later run can tell its output apart. Write
                     <project>/.beadsmith/COVERAGE.md: commitment # → bead ID.
Phase 5  VALIDATE    `bv`: cycle check must be clean; read the triage order
                     and confirm it tells a buildable story (foundations
                     surface first, capstones last). Wrong story = missing or
                     backwards edges; fix and re-run.
Phase 6  GATE        Smart-model review of the draft graph (NON-SKIPPABLE):
                     does each bead stand alone? Right granularity? Deps real?
                     Then diff the graph against the Phase 1 list: every
                     commitment traced, nothing oversimplified, nothing
                     silently dropped.
Phase 7  POLISH      2–3 refinement passes over the beads in plan-space.
                     Each pass: re-read the plan, walk every bead, tighten
                     wording, fix deps, split or merge where sizing was off.
                     Hard constraints every pass: DO NOT OVERSIMPLIFY ·
                     DO NOT DROP FEATURES/FUNCTIONALITY · test + logging
                     tasks stay in. Stop when a pass changes nothing.
Phase 8  HANDOFF     Report ready-front bead IDs for the swarm and stop. The
                     graph builds under the green ratchet from here. Stack
                     note for executors: `br` has NO `comment` subcommand;
                     closure evidence goes in the commit message or the bead
                     description, nowhere else.
```

## The self-contained bead body (Phase 4 template)

Each bead must read like a brief a contractor could execute with zero context.
No "see plan doc", because after this run there is no plan doc:

```
WHAT      one-sentence deliverable
WHY       background + rationale — which plan commitment it serves, what
          problem, why now
HOW       approach + key considerations / gotchas specific to THIS codebase
DEPENDS   what must land first (mirrored as real br deps, not just prose)
DONE WHEN observable success criteria — including the test that proves it and
          what its logging must show
```

## Contract (what orchestrators can rely on)

kickstart, project-autopilot, and ux-maximizer consume this skill as their
plan→tracked-work stage ("plan-workflow → beadsmith → build under the green
ratchet"). The interface is fixed:

- **Input:** a plan/decision artifact path. beadsmith never invents scope.
- **br:** the operational output. All created beads labeled `beadsmith`.
- `<project>/.beadsmith/COVERAGE.md`: commitment → bead-ID trace (the audit
  trail; proof nothing was dropped; safe to delete after the run).
- Exit state: `bv` clean (no cycles), polish passes complete, ready front
  reported. **Nothing implemented**: beadsmith ends where execution begins.
- Stack note baked into the handoff: `br` has no `comment` subcommand. Put
  closure evidence in the commit message or bead description.

## THE EXACT PROMPT

```
/beadsmith
Plan:    <path to the plan / decision artifact>
Project: <project path, default cwd>
Label:   <extra bead label, default just "beadsmith">
```

## Anti-Patterns

- Ideating. New ideas surfaced mid-conversion go to the decision inbox or a
  later idea-forge run, never quietly into the graph.
- Converting a vague wish. No agreed plan, no beadsmith. Bounce upstream.
- Skipping the Phase 0 inventory. You'll duplicate open beads and re-plan
  shipped work.
- Beads whose body says "see the plan doc." The graph replaces the doc;
  self-contained or it doesn't ship.
- One mega-bead per plan section, or dust-sized beads no agent can commit
  green. Both fail the one-session sizing test.
- DEPENDS as prose only. Every dependency in a body must exist as a real br
  edge, or the swarm builds in the wrong order.
- Reaching for `br comment`. It doesn't exist; use the commit message or bead
  description.
- Declaring done without `bv` clean and the coverage diff. "Looks complete"
  is how the three missing commitments stay missing.
- Polish passes that "simplify" by dropping features or deleting test/logging
  tasks. Polish tightens wording and deps, never scope.
- Starting implementation. Hand the ready front to the swarm; plan-space ends
  here.

## References

- [GRAPH-CHECKS.md](references/GRAPH-CHECKS.md): br/bv command crib, the
  sizing test expanded, and the per-pass polish checklist
