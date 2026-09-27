# Forge: canonical kickoff prompt

Fill in the <SLOTS> and paste to launch a full idea→ship-ready run. It sets the
standing doctrine once (the run's constitution), then forge advances staged: one
stage, one move at a time. To run it unattended across days, wrap with /goal (if your
harness has it) using the clearable terminal condition at the bottom.

```
MISSION
Build <WHAT YOU'RE BUILDING> and carry it to SHIP-READY.
Target: <PROJECT PATH, or "a new project at ~/<NAME>">.
Use /forge to detect the stage and drive the right skill the whole way. Do NOT do it
all at once; forge is a successive-prompter: advance one stage and one move at a time.

INTENT (the ceiling for "done" and "full")
<ONE PARAGRAPH: who it's for · the one job it does · what winning looks like ·
commercial vs fun · the keep-it-small scope. If unsure, draw this out first via
reality-check intent extraction and confirm it with me before any deep build.>

OPERATING DOCTRINE (standing rules for the entire run)
- Successive-prompter: one stage, one move per tick; issue the next instruction only
  after the last is verified green. Never dump or parallelize the whole plan.
- Green ratchet: every batch ends with build + tests + smoke passing, or revert to the
  last checkpoint. Checkpoint-commit after each green batch.
- Decision inbox: never make my judgment calls (pricing, scope cuts, brand,
  architecture bets, anything outward-facing or irreversible). Queue them in
  <project>/.autopilot/DECISIONS.md with a recommended option, and work around them.
- Anti-overbuild (load-bearing): ideate the LEANEST path, not the most maximal; a
  walking skeleton that runs end-to-end before any deep build; the Intent above is the
  ceiling; beyond-intent ideas go to a `someday` backlog, never built; idea-forge (or
  idea-wizard, if installed) ONLY the non-obvious, high-leverage forks (everything else gets the boring default);
  no abstraction until 3 real call sites; pair every "build" with a "deliberately skip."
- Smart-model review: re-verify every audit/subagent finding against the real code
  before acting; finders over-report; kill false positives before they cost a commit.
- Verify by running: prove behavior by running it, not by "tests pass." For UI, look at it.
- Stage gates: optimization (profile-first) and the testing-posture push run only once
  features are settled; guard idea-forge / idea-wizard mass-dispatch on late-stage code (>10 modified
  files OR >3 in-flight beads → ask me first).

THE ARC (forge detects the stage and drives; you don't pick)
  bare idea          → /kickstart         (lean scaffold + intent + beads)
  working scaffold   → /project-autopilot  (to done; it routes to /bug-zero, /perf-max,
                                            /ux-maximizer as its lenses fire)
  done + delivers    → /ship-it           (pre-flight everything reversible; STOP at the
                                            outward buttons)

EXECUTION BACKEND (detect & degrade, never block)
If ntm is installed, prefer an ntm + Codex swarm (~4 cod + 2 cc; CC controller running
its looper ~every 4 min; bv-prioritized assignment; rch for builds if installed, else
build locally / via CI; recover stalled beads; clear stale artifacts). If ntm or Codex
is unavailable or rate-limited, degrade: caam (if installed) to swap accounts → CC-only swarm →
in-process subagents → solo sequential (always available: the floor). A lapsed Codex
sub or a missing swarm tool NEVER stops the run; it just slows it.

BUDGET & CADENCE
Token budget: <e.g. "stop and report at +500k", or "no cap">. Checkpoint-commit every
green batch. Append a one-line progress note to the project's own run log (e.g.
<project>/.autopilot/LOG.md) each tick. Push long builds/CI to background monitors and
keep working in parallel.

TERMINAL STATE (the goal, reachable WITHOUT me)
SHIP-READY: green · delivers the Intent · release fully pre-flighted (version, changelog,
artifacts, checksums, preview deploy, drafted release) · with EVERY outward ship button
and EVERY judgment decision parked in the inbox. Stop there, surface the inbox + the
buttons, and hand me a one-screen report. Do NOT press publish / prod-deploy /
store-submit / cargo-publish / push. Those are mine.

FIRST MOVE
Read AGENTS.md + README + any plan/notes docs fully; deep-investigate the code. Place
the stage. Write/confirm the Intent Statement. Then begin the first move of the current
stage: one step, verified green, before the next.
```

## To run it unattended across days

1. Paste the filled-in prompt to kick off.
2. Wrap persistence with **/goal** (if your harness has it; otherwise re-paste the
   prompt each session) using a CLEARABLE condition (ship-ready is reachable
   without you; "ship it" would never clear and the hook would spin):
   ```
   /goal forge <project> to SHIP-READY: green, delivers intent, release pre-flighted,
   with all ship buttons and decisions parked in .autopilot/DECISIONS.md. Do not press
   any outward button.
   ```
3. Set cadence with **/loop** (self-paced) or an **ntm looper** for a swarm.
4. Watch the **decision inbox**. When it grows, that's your signal it needs you.
