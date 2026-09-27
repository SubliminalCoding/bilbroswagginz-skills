# Prompts: Kickstart

The staged prompts, one at a time. The defining one is Phase 1: it's idea-forge
*inverted*, pointed at the leanest path, not the wow-factor maximum. (Contrast
ux-maximizer/references/PROMPTS.md, which is calibrated for ambition-toward-more. Here, ambition
is toward elegant subtraction.)

## Phase 0: idea → Intent Statement (the ceiling)

```
here's the general idea: [idea]. before any building, draw out a crisp Intent
Statement: who it's for, the one job it does, what "winning"/"done enough" looks like,
commercial vs fun, and the keep-it-small scope. this is the CEILING: "full" means
delivers THIS, nothing past it.
```

## Phase 1: the lean-path idea-forge pass (the inverted wow prompt)

```
run idea-forge on the SIMPLEST, most elegant way to BUILD [idea] that fully
delivers [intent]. optimize for the leanest path: the least code, the fewest moving
parts, the most boring proven choices that still nail it. dig deep for cleverness that
REMOVES work, not adds it. the goal is a tight working thing, not a maximal one.

give me the top 5 build approaches AND, just as important, a bottom 5: what we can
NOT build, defer, or delete and still deliver the intent. then recommend the leanest
approach that hits the intent.
```

Genuinely big/expensive/hard-to-reverse fork (core stack, data model, architecture)?
→ dueling-idea-forge for adversarial competing takes before committing. New stack?
→ research-software (if installed; otherwise read the stack's docs).

## Phase 2: walking skeleton

```
build the THINNEST possible version that runs end-to-end (one real path through the
whole thing) plus a single green test. no features beyond that yet. we want proof the
shape works and a baseline to ratchet from before we invest.
```

## Phase 3: selective recursion (the gate that prevents overbuild)

For each feature, decide FIRST whether it even needs ideation:

```
is the best way to build [feature] non-obvious AND high-leverage or costly to reverse?
  - NO  → just build the simplest version that delivers the intent (YAGNI). no ideation.
  - YES → run a lean-path idea-forge pass on JUST this fork (simplest elegant option +
          what to skip), pick, then build.
```

Cap it: at most a couple of ideated forks per round. Most features get built, not
ideated. Then: planning-workflow (if installed) → beadsmith → build under the green ratchet.

## Between rounds: creep check

```
are we still on the smallest path that delivers the intent, or has scope crept? list
anything we've built or are about to build that the Intent Statement doesn't require and
move it to a `someday` backlog. cut, don't power through.
```

## Phase 4: handoff

```
we have a walking, intent-aligned scaffold + green baseline + a bead backlog. hand off
to /project-autopilot to take it from 1→done. kickstart's job is done at the scaffold.
```
