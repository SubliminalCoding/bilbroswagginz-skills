---
name: skillify-harvest-loop
goal: Convert recurring agent friction into registered, cross-runtime skills before it re-derives itself
cadence: weekly
run_via: /loop or /schedule (weekly)
routes_to: [skillify, session-mine, memory-forge]
stop_condition: Every above-bar candidate from the window is skillified+registered or logged as a project lesson; unharvested backlog hits zero
verification_gate: A skill counts only if the registry shows it installed AND (for a global skill) the cross-runtime symlink + shared-memory note exist
---

# Skillify Harvest Loop

**Goal.** Skills should compound, but mid-flow, wins go uncaptured.
This loop keeps your agents compounding instead of re-deriving the same
fixes.

**Each iteration.**
1. Mine the window's sessions (`session-mine`) and session logs for the four skill-worthy signals: a debugging breakthrough, an architecture pivot, a gotcha that ate real time, or an external-system workaround.
2. Apply the worthiness test: *would a future session, or your other agent runtimes, on a DIFFERENT project benefit?* One-offs go to a project-lessons note; generalizable methods get promoted.
3. For each promoted win: run `skillify`, then register it in your skill registry (project scope by default) and note it in your shared agent-memory file.

**Stop when** every above-bar candidate from the window is either skillified+registered or explicitly logged as a one-off lesson: the "unharvested generalizable wins" backlog is zero for the period.

**Verification gate.** A win is only "harvested" if the registry shows it installed AND, for a global skill, the cross-runtime symlink + shared-memory note exist. A drafted `SKILL.md` that isn't wired in is FAILED. Enforces the "symlink, never copy, and record it" rule.

## Copy-paste prompt
```
/loop Harvest skill-worthy wins from my recent work. Each pass: mine recent sessions
(session-mine) and session logs for a debugging breakthrough, architecture pivot,
time-eating gotcha, or external-system workaround. Apply the test "would a future
session, or my other agent runtimes, on a DIFFERENT project benefit?" One-offs
become a project-lessons note, generalizable methods get skillified via the skillify
skill and registered in the skill registry, then noted in the shared agent-memory
file. A skill only counts when the registry shows it installed AND, for a global skill,
the cross-runtime symlink + shared-memory note exist. Stop when no unharvested above-bar win remains
in the window.
```

**Cadence / notes.** Weekly is the sweet spot: long enough to accumulate wins, short enough that context is still fresh in your session history. `memory-forge` curates what the mine unearths; `skillify` promotes the methods that generalize.
