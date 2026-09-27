---
name: problem-breakthrough
description: >-
  Crack a STUCK problem with structured creative reframing, not more brainstorming.
  Generate a few genuinely orthogonal solution approaches, lead with the reframe
  that dissolves the blocker, rank by feasibility x leverage, and recommend one.
  Use when stuck on a specific blocker, "this approach isn't working", "we've
  tried everything", "how else could we do this", "think outside the box",
  "creative solution", "we're blocked on X", "there has to be a better way",
  "crack this". NOT for project-improvement ideation or feature backlogs; that's
  idea-forge. This is convergent (one problem, best path); idea-forge is
  divergent (many features → beads).
---

# problem-breakthrough

> **Core insight:** A stuck problem is almost never solved by trying the current
> approach harder. It's solved by a REFRAME: questioning a constraint you've
> been silently accepting. Find the reframe first; the solution falls out of it.

## When to use this vs idea-forge

| problem-breakthrough (this) | idea-forge |
|------------------------------|-------------|
| ONE blocker, need the best path | A project, need a backlog of improvements |
| Convergent: 3–5 orthogonal options → recommend 1 | Divergent: 30 ideas → winnow → beads |
| "This isn't working / we're stuck" | "What should we build next" |
| Output: a reframe + a chosen approach + a spike | Output: prioritized beads |

If you catch yourself wanting to generate features or a backlog, you want
idea-forge. If you're banging on one wall, you want this.

## The process

### 1. State the problem AND the constraint you've been accepting
Write the blocker in one line. Then write the **hidden assumption**: the thing
you've been treating as fixed. ("We must get the live auth token." "It has to go
through the API." "We can only run this on one machine.") The constraint is
usually where the breakthrough lives.

### 2. Run the reframe moves (the heart of this skill)
For each, ask the question out loud and see what opens up:

| Move | The question |
|------|--------------|
| **Question the constraint** | Why do we even need X? What if we didn't? |
| **Change the layer** | Can we solve this somewhere else in the stack? (API → storage → UI → network → OS) |
| **Invert** | What would make this trivial? What would make it impossible, then avoid that? |
| **Remove the requirement** | Is the hard part actually required, or inherited? |
| **First principles** | Strip the accumulated how; what is the bare goal? |
| **Borrow** | Who in an adjacent domain already solved this shape? |
| **Read vs fight** | Is there already a byproduct/artifact that has what we want? (cache, log, store, export) |
| **Shrink** | Solve a 90% version that's 10% the work; is 90% enough? |

### 3. Generate 3–5 ORTHOGONAL approaches
Genuinely different, not variations of one idea. If two options share the same
assumption, collapse them and find a third that breaks it. Force at least one
option that drops the constraint from step 1 entirely.

### 4. Rank by feasibility x leverage; pressure-test the favorite
Score each on can-we-actually-do-this × how-much-it-unlocks. **Demote the
"brute-force the current approach" option** unless it genuinely wins. For the top
pick, name the single riskiest assumption and how to **spike** it cheaply (a
5-minute probe, not a full build) before committing.

### 5. Recommend one + name the reframe
State the chosen approach, the reframe that unlocks it, and the spike to de-risk
it. One paragraph, not a menu.

## Worked example (a browser-extension message-detection rebuild)
- Blocker: detect new chat messages in a web chat app via a browser extension.
- Accepted constraint: *"we need the chat service's live auth token."*
- Reframe (question the constraint + change the layer + read-vs-fight): *"Do we?
  The chat app already downloaded the messages and saved them to IndexedDB."*
- Orthogonal options: (a) chase token via refresh flow, (b) intercept token off
  the wire, (c) **read the app's IndexedDB store directly**: no token at all.
- Winner: (c). Spike: dump the IndexedDB db/store names to confirm the data's
  there (it was). Hours of token-fighting dissolved.

## Anti-patterns

| Don't | Do |
|-------|-----|
| Brainstorm 30 variations | 3–5 ORTHOGONAL approaches |
| Optimize the stuck approach harder | Reframe the constraint first |
| Skip naming the hidden assumption | It's where the breakthrough hides |
| Present a menu and stop | Recommend one + the reframe + a spike |
| Commit before spiking the risky assumption | 5-min probe beats a wrong full build |
| Reach for idea-forge on a blocker | Wrong shape; use this |

## Checklist
- [ ] Blocker stated in one line + the accepted constraint named
- [ ] Ran the reframe moves; found at least one that drops the constraint
- [ ] 3–5 genuinely orthogonal approaches (not variations)
- [ ] Ranked by feasibility x leverage; brute-force option demoted unless it wins
- [ ] Recommended ONE, named the reframe, defined a cheap spike for the risk
