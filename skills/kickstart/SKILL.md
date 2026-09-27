---
name: kickstart
description: >-
  Take a project from a general idea to a working, intent-aligned scaffold by
  ideating the LEANEST elegant way to build it (not the most maximal), standing up a
  walking skeleton that runs end-to-end first, then filling out only what the intent
  needs via selective idea-forge passes on the forks that actually matter, and handing
  off to project-autopilot once a green baseline + bead backlog exist. Use when
  "start a new project", "build this from scratch", "greenfield", "scaffold this
  idea", "0 to 1".
---

# Kickstart

The 0→1 vertical: the one project-autopilot refuses to do (it requires a scaffold
already standing). kickstart builds that scaffold from a general idea, then passes the
baton. Bookend of ship-it: *kickstart it … ship it.*

> **Core Insight:** A greenfield ideation pass pointed at "more" is how projects drown
> before they run. So kickstart inverts the ambition: its ideation optimizes for the
> *leanest elegant path to the intent*: the least code, the fewest moving parts, the
> boring proven choices, and treats *not building* as a first-class output. It gets
> to a walking, intent-aligned scaffold fast, then hands off. The whole skill is a
> brake on overbuilding, because that's the failure mode it exists inside.

## Anti-overbuild gates (the spine; these override enthusiasm)

1. **Lean-path ideation, not maximal.** kickstart's idea-forge pass optimizes for the
   simplest elegant build that delivers the intent (what to NOT build), not feature
   maximalism. (This is the OPPOSITE calibration from ux-maximizer's wow prompt; the
   wow pass comes later, if ever. See references/PROMPTS.md.)
2. **Walking skeleton first.** The thinnest end-to-end runnable thing + one green test
   before any deep build. Proves the shape, gives the ratchet a baseline, and means
   stopping early still leaves something that works.
3. **Intent is the ceiling.** "Full" = delivers the Intent Statement. Beyond-intent
   ideas go to a `someday` backlog, never auto-built. Scope ratchet: close the gap to
   intent, never widen it.
4. **Selective recursion + ideation budget.** Run idea-forge on a sub-fork ONLY when it's
   non-obvious AND high-leverage / costly-to-reverse. Obvious, cheap, reversible
   decisions → pick the boring default (YAGNI). Cap ideated forks per round; most
   features just get built.
5. **Subtraction pass.** Every architecture/feature decision pairs a "build" top-N
   with a "deliberately skip/defer" bottom-N. Non-building is an explicit deliverable.
6. **Rule of three.** No abstraction, framework, or config layer until 2–3 real call
   sites actually need it. No speculative generality, no "we might want it later."
7. **Hard handoff.** Stop at a walking scaffold + green baseline + bead backlog and
   hand to project-autopilot. kickstart is NOT the 1→done loop; it builds the runway.
8. **Creep check.** Between build-out rounds: "are we still on the smallest path that
   delivers the intent?" Catch sprawl early; cut, don't power through.

## The Pipeline (staged: one step at a time)

```
Phase 0  IDEA → INTENT    Turn the general idea into a crisp Intent Statement (who it
                          serves, what job, what "winning" is, commercial-vs-fun, and
                          the keep-it-small scope). This is the CEILING for "full".
Phase 1  HOW (lean-path)  idea-forge the simplest elegant way to BUILD it → top 5
                          approaches PAIRED with a bottom-5 to deliberately skip. A
                          genuinely big/expensive architecture or stack fork →
                          dueling-idea-forge. Choose the leanest that delivers the intent.
                          (research-software, if installed, to learn a new stack.)
Phase 2  WALKING SKELETON Scaffold the thinnest thing that runs end-to-end + one green
                          test. Checkpoint. NOW the green ratchet engages.
Phase 3  BUILD-OUT        Feature by feature toward the intent: idea-forge ONLY the
                          non-obvious high-leverage forks (gate #4), YAGNI default for
                          the rest, planning-workflow → beadsmith → build under the
                          ratchet. Creep check between rounds. Fill ONLY to the intent.
Phase 4  HANDOFF          When a working, intent-aligned scaffold + green baseline +
                          bead backlog exist → hand to /project-autopilot (1→done),
                          which later routes to the quality verticals and /ship-it.
```

## Skills it orchestrates

idea-forge (the recursive engine, pointed at leanness) · dueling-idea-forge (big
architecture/stack forks only: adversarial takes before committing) ·
planning-workflow (chosen approach → plan; if not installed, write the plan doc directly) ·
beadsmith (plan → beads) · research-software (learn a new stack; if not installed, read the stack's docs) · reality-check
(intent extraction in Phase 0) → then **project-autopilot** at the handoff. Staged
prompts in [references/PROMPTS.md](references/PROMPTS.md).

## Shared mechanisms (inherited from project-autopilot)

Green ratchet (engages at the walking skeleton), decision inbox, successive-prompter
timing, as in project-autopilot. **Lifecycle position:** kickstart (0→1) →
project-autopilot (1→done) → bug-zero/perf-max/ux-maximizer (quality) → ship-it (out).

## THE EXACT PROMPT

```
/kickstart
Idea: <the general thing you want to build>
Intent: <who it's for / what winning looks like, if you know it; else Phase 0 draws it out>
Mode: solo (default) | swarm (optional; needs ntm installed)
```

## Anti-Patterns

- Ideating for "more" instead of for the leanest path that delivers the intent.
- Deep-building before a walking skeleton runs end-to-end (no baseline, no proof).
- Running idea-forge on every facet: selective recursion is the whole point (gate #4).
- Building past the intent because an idea was cool: that's a `someday` backlog item.
- Abstracting/frameworking before the rule of three is met.
- Trying to take it all the way to done. Hand off to project-autopilot at the scaffold.
- Skipping the subtraction pass: what you DON'T build is half the design.

## References

- [PROMPTS.md](references/PROMPTS.md): the lean-path idea-forge prompt, subtraction pass, recursion gate, creep check
