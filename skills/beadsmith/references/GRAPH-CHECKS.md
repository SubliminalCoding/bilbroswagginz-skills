# Beadsmith: Graph Checks & Command Crib

## br command crib (the subset beadsmith uses)

```bash
br list --status=open            # existing planned work (merge, don't duplicate)
br list --status=closed          # what already shipped (don't re-plan it)
br create "Epic: <title>" --type=epic --label=beadsmith,<plan-slug>
br create "<task title>" --deps=<epic-id>,<blocker-id> --label=beadsmith,<plan-slug>
br dep add <task-id> <blocker-id>   # add an edge after creation
br update <id>                   # fix a body during polish passes
br show <id>                     # read one bead back during walks
```

Hard fact: **`br` has no `comment` subcommand.** Closure evidence (test output,
links, "verified by X") goes in the commit message or gets appended to the bead
description. Do not invent a comment flow; agents downstream will copy it.

## bv validation (Phase 5)

```bash
bv          # cycle check + triage order
```

Two distinct checks, both required:

1. **No cycles.** A cycle means the outline reasoning in Phase 3 failed. Fix
   the edges, don't delete a node to break the loop.
2. **The triage order tells a buildable story.** Read it top to bottom as "what
   the swarm does first." Schema/scaffold/foundation beads should surface
   early; integration and capstone beads late. A capstone surfacing first means
   its dependencies were never wired. The cycle check alone won't catch a
   MISSING edge; only the story-read does.

## The sizing test (Phase 2, expanded)

One task = one focused agent session ending in a commit that keeps the build
green. Apply it concretely:

- Can't describe DONE WHEN in 1–3 observable criteria → too big, split.
- The split pieces couldn't each land green on their own → split along the
  wrong seam; split by layer or by vertical slice instead.
- An epic with one task → flatten to a task.
- A subtask is only born when a task body genuinely exceeds one sitting;
  subtasks are pressure relief, not default structure.
- Per epic, TEST and LOGGING tasks are explicit nodes depending on the epic's
  feature tasks. If a polish pass is tempted to fold them "into" the feature
  beads, that's the oversimplify failure mode: refuse.

## Coverage diff (Phase 6)

`.beadsmith/COVERAGE.md` format: one row per Phase 1 commitment:

```
| # | Plan commitment (quote or tight paraphrase) | Bead | Status |
|---|---------------------------------------------|------|--------|
| 1 | "rate-limit the webhook endpoint"           | br-41| traced |
| 2 | "migrate legacy rows before cutover"        | br-44| traced |
| 3 | "admin can export CSV"                      | —    | DROPPED — resolve |
```

The gate fails while any row is DROPPED. Legitimate resolutions: create the
missing bead, merge into an existing open bead (note its ID), or record an
explicit operator-approved descope in the decision inbox. "It's implied by br-41"
is not a resolution. Make it a DONE WHEN line in br-41's body, then mark it
`merged:br-41`.

## Per-pass polish checklist (Phase 7)

Each pass, walk EVERY bead and ask:

1. Standalone? Could a contractor with zero context execute from the body alone?
2. Sized right? Still passes the one-session green-commit test?
3. Edges real? Every prose DEPENDS mirrored as a br edge; no vibes-edges added?
4. Scope intact? Nothing simplified away; test + logging tasks still present?
5. Still true? Does the bead match the plan AND the codebase as they are now?

Stop at 3 passes or when a pass changes nothing, whichever comes first.
