---
name: forge
description: >-
  The one-handle lifecycle conductor: take a bare idea OR a project at any stage and
  carry it all the way to shipped, by detecting where it is and driving the right
  skill at the right time: kickstart (0→1) → project-autopilot (1→done, which itself
  routes to bug-zero/perf-max/ux-maximizer) → ship-it (out the door). Pauses at every
  judgment gate. Use when "take this idea all the way", "build and ship this", "run
  the whole lifecycle", or you don't want to think about which skill to reach for.
---

# Forge

One handle, all the blades. You don't pick the vertical; forge detects where the
project is in its life and runs the right specialist, advancing stage by stage from a
bare idea to a shipped product, pausing only at the judgment gates that are yours.

> **Core Insight:** This is a *conductor, not a merge.* It adds almost no logic of its
> own: it's a stage detector + a macro-loop over seven skills that stay exactly as they
> are and remain independently callable. That's deliberate: fusing their contents into
> one file would be the ~1,000-line overbuild this whole family exists to prevent. The
> value here is a single entry point and correct *sequencing*, nothing more. **Every
> safety brake lives in the sub-skills, so forge inherits them all and can bypass
> none**: kickstart's overbuild gates, autopilot's green ratchet + decision inbox +
> late-stage guard, the verticals' gates, ship-it's never-auto-publish discipline.

## The lifecycle state machine

```
DETECT   Where is this? Use reality-check + the shipped-reality probe to place it:
           • bare idea, no working scaffold          → STAGE 1
           • working scaffold, not yet delivering intent → STAGE 2
           • green + delivers intent, ready to release   → STAGE 3
           • shipped                                  → DONE
           • ON FIRE in production (any stage)        → /incident-zero FIRST:
             stabilize → evidence → fail-first regression → prevention, then
             re-DETECT and resume the lifecycle where the fire interrupted it

ROUTE    Run the stage's skill (each owns its own loop, gates, and convergence):
           STAGE 1  /kickstart        idea → walking, intent-aligned scaffold + beads
                    exit: green baseline + intent + bead backlog exist
           STAGE 2  /project-autopilot  scaffold → done (it routes to /bug-zero,
                    /perf-max, /ux-maximizer as its lenses fire)
                    exit: intent satisfied (dry passes + final reality-check pass)
           STAGE 3  /ship-it          pre-flight everything reversible, hand the user the
                    outward buttons (publish/deploy/submit): ask-user
                    exit: the user ships + post-ship smoke passes

ADVANCE  When a stage's exit condition holds, re-DETECT and move to the next.
PAUSE    At ANY ask-user gate (ship buttons, pricing/scope/release decisions, a
         late-stage idea-forge / idea-wizard run): surface to the user, never guess. Decision inbox carries
         everything that's waiting; forge works around it where it can.
DONE     Shipped (or the user stops it, or a sub-skill's budget caps the run).
```

## What forge adds (and deliberately does not)

**Adds:** one entry point · stage detection · correct sequencing · the 0→1 entry and
the ship exit that no single existing skill spans.
**Does NOT add:** any quality/fix/build logic (that's the verticals), and any new
risk: it can only invoke the sub-skills, which keep all their own brakes.

## Preconditions

Just an idea or a project path. forge figures out the stage. (If you already know the
project needs only ONE thing (just bugs, just a perf pass, just shipping), call that
skill directly; forge is for carrying the *whole arc*, not for single-stage work.)

## THE EXACT PROMPT

```
/forge
Target: <a bare idea (text) | a project path at any stage>
Goal:   <"idea → shipped" (default) | or a stage to stop at, e.g. "to done, don't ship">
Mode:   solo | swarm
```

## The family it conducts

| Stage | Skill | Owns |
|---|---|---|
| 0→1 | /kickstart | lean greenfield scaffold (overbuild-gated) |
| 1→done | /project-autopilot | the diagnose→route→loop engine; routes to ↓ |
| correctness | /bug-zero | (via autopilot's correctness lens) |
| performance | /perf-max | (via autopilot's perf lens) |
| UI/UX | /ux-maximizer | (via autopilot's UX lens) |
| ship | /ship-it | reversible pre-flight + the outward buttons |
| prod incident | /incident-zero | (via DETECT's fire branch) stabilize-first + prevention-before-close |

## Anti-Patterns

- Merging the sub-skills' contents instead of delegating. forge is a router; a merge
  is the overbuild the family exists to prevent.
- Skipping a stage's exit condition to rush ahead (each stage hands off clean or not
  at all).
- Auto-pressing ship buttons or answering decision-inbox items. Inherited ask-user.
- Using forge when the project needs only one stage. Call that vertical directly.
- Re-implementing a brake here that a sub-skill already owns. Inherit, don't copy.
