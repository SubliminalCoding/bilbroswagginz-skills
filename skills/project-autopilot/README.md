# project-autopilot: overview map

A one-page map of the skill, so you can see the whole thing at a glance without
reading all ~835 lines. (This README is documentation *about* the skill; the skill
itself is SKILL.md + the six reference files.)

## What it is, in one breath

Point it at a project already in motion. It extracts what the project is trying to
be, diagnoses the gaps through expert lenses, routes each gap to the best installed
skill, and executes in small green-verified batches, queuing anything that needs
your judgment into a decision inbox instead of guessing. It's **your universal
in-flight prompt, generalized and automated.**

There is **no executable code**; it's instructions. The logic lives in the ~100
skills it routes to and the model reading it. It's a conductor, not a blade.

## The three pillars (why autonomy is safe)

1. **Stated intention:** a one-paragraph Intent Statement it measures everything
   against, so it optimizes toward *your* goal, not generic best practice.
2. **Green ratchet:** every batch ends with build+tests+smoke passing or gets
   reverted. The loop can never make things worse.
3. **Decision inbox:** judgment calls (pricing, scope, releases, deploys) queue
   for you; the loop works around them and never answers its own.

## The fourth idea (the soul): successive-prompter

Its real value is **timing**, not just routing. It withholds each phase's
instruction until the prior one finishes, so the model focuses on one thing at a
time, the same way you prompt at each level by hand. One move per tick.

## The tick loop

```
ASSESS → DECIDE → ROUTE → EXECUTE → VERIFY → LOG → CONVERGE?
```
Diagnose what's true now → pick ONE move (the way the operator would) → map it to a skill
(auto or ask-user) → run a small batch → must end green or revert → checkpoint +
log → done? else tick again. Resumable from beads + inbox + checkpoints alone.

## The files

| File | Holds | Read it when |
|---|---|---|
| **SKILL.md** | The controller: preconditions, Phase 0 launch, the tick loop, the done report | Always; it's the entry point |
| **EXECUTION.md** | The big-skill pipeline (staging heavy audits) + the execution-backend ladder (detect & degrade) | EXECUTE step, heavy skills or backend choice |
| **LENSES.md** | The 9 diagnostic facets + relevance filter + the shipped-reality probe + re-verify rule | DECIDE step, when diagnosing |
| **SKILL-ROUTING-MAP.md** | gap type → installed skill, each marked auto \| ask-user | ROUTE step, picking the tool |
| **OPERATOR-MOVES.md** | 19 operator steering moves, mined from real session history, with paraphrased examples | DECIDE step: "what would the operator do here?" |
| **CONVERGENCE.md** | Green ratchet, done criteria, saturation gate, optimization-pass window, budgets | CONVERGE step, deciding done |
| **SAFETY-RAILS.md** | Batch limits, memory guards, swarm-tending, forbidden actions, escalations | Before every EXECUTE |

## The stage gates (what runs when)

- **Early build:** correctness > everything; fix-broken-first; small batches.
- **Maturity (features settled):** the optimization pass (profile →
  repeatedly-apply optimize-proof) and the testing-* posture push,
  gated here, *never* on a still-churning build.
- **Convergence:** review until **saturated** (diminishing bugs per token, not
  zero); when saturated but more work is suspected, re-arm with a fresh analysis
  sweep → new beads. Stop on K=2 dry sweeps + final reality check.

## Execution backend: detect & degrade

The swarm is an accelerator, never a requirement. At implement time it probes what's
live and picks the best rung: ntm+Codex swarm → ntm+CC swarm → other-model / native
parallel sub-agents (e.g. a runtime's delegate-task tool) → in-process Agent/Workflow →
**solo CC sequential (the floor)**. A lapsed Codex sub or a missing swarm tool never
blocks the loop.

## How to invoke

```
/project-autopilot
Target: <project path>   Mode: solo | swarm   Budget: <max ticks, default 25>
```

## Status

- **Built:** 2026-06-05. ~835 lines across 7 files (+ this README), 0 executable code.
- **Piloted:** on a game project (2 ticks): the late-stage guard caught a large
  uncommitted working tree, and the run found and fixed an unauthenticated endpoint.
- **Candidate:** the pack's v2 flagship. OPERATOR-MOVES examples are paraphrased
  from real sessions; project names and identifying details have been removed.
