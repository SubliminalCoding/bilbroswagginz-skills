---
name: reality-check
description: >-
  Hold a project's stated vision up against what's actually built and running,
  and report the gap without flattery: what works, what's claimed-but-broken,
  what's missing, what drifted, ending in an explicit SHIP-READY / NOT verdict.
  When a project has no intent doc at all, flips to INTENT EXTRACTION instead:
  reconstructs the project's goals from code, tests, beads, and commits into a
  confirmable intent statement. Use when "reality check", "where are we really",
  "does this actually work", "are we on track", "gap analysis", "what's missing",
  or when project-autopilot / ship-it / idea-forge need their assess, preflight,
  or intent-extraction step.
---

# Reality Check

Every project tells two stories: the one its docs tell and the one its runtime
tells. This skill reads both and reports the difference. It is the instrument
the other verticals steer by (project-autopilot's assess step, ship-it's
Phase-0 preflight gate, idea-forge's intent source of last resort), and its only
loyalty is to what actually ran. The value is catching "we think it works but it
doesn't" *before* a ship, a demo, or a week of polish built on sand.

> **Core Insight:** The README is a press release; the only admissible evidence
> for "it works" is watching it work. Build it, launch it, exercise the headline
> promise end to end, unmocked. The gap that kills projects is never the one in
> the backlog (that one's at least known). It's the one between "we believe it
> works" and "we watched it work." Every WORKS in the report cites an execution
> trace, never a doc.

> **Core Insight #2:** Reality-check is **read-only by design**. It never fixes,
> never refactors, never "while I'm here"s. The moment the assessor starts
> repairing, the report stops being trustworthy (you grade what you wish you'd
> found). Output is a report, a verdict, and inbox items. Fixing belongs to
> project-autopilot and the quality verticals downstream.

## Preconditions

1. **A project directory with code in it.** That's the whole bar: this skill
   exists precisely for projects whose docs are missing, stale, or lying.
2. **br is optional.** If `br list` works, the bead graph (open AND closed) is a
   first-class evidence source; if not, proceed without it and say so in the
   report. Never fail the run over a missing tracker.
3. **Leave the tree as found.** Builds/test runs/launches must not dirty the
   working tree (stash or clean anything they create); downstream skills ratchet
   off a clean baseline. On a unified-memory box (e.g. an NVIDIA DGX Spark/GB10)
   with `memguard` installed, launch anything memory-heavy via `memguard run`,
   never bare; elsewhere, launch normally.

## Mode gate

Intent sources, in priority order: AGENTS.md · README.md · CLAUDE.md · plan
docs · any project notes in the repo · existing
`<project>/.autopilot/INTENT.md`. Any of these states what the project is trying
to be → **ASSESS** (default). None do → **EXTRACT** first, then (on a combined
run) feed the confirmed statement back into ASSESS.

## ASSESS Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  PROMISES   Read every intent source. Distill into a NUMBERED promise
                    list: concrete claims the project makes about itself ("does
                    X", "handles Y", "deployed at Z"), headline promises flagged.
                    This list is the rubric; everything after grades against it.
Phase 1  LEDGER     The tracker's own account: `br list --status=open` and
                    `--status=closed`, `bv` (if installed) for triage shape.
                    Closed beads = what the project claims it finished; open =
                    what it admits is missing. Note promises with no bead
                    anywhere; invisible work is the usual hiding place for drift.
Phase 2  RUN IT     The heart of the skill. Build · test · launch · exercise each
                    headline promise end to end as a user would (memguard-wrapped
                    on a unified-memory box if you have it; bare elsewhere).
                    Record verbatim what ran, what errored, what
                    was unreachable. A green test suite is supporting evidence;
                    only an unmocked end-to-end exercise proves a headline
                    promise. Capture the actual commands + output traces.
Phase 3  CLASSIFY   Per promise, one verdict with evidence cited
                    (references/CLASSIFICATION.md for the rubric):
                    WORKS · PARTIAL · CLAIMED-BUT-BROKEN · MISSING · DRIFTED
                    (built something other than promised), plus UNDOCUMENTED
                    for real built capability no doc ever promised.
Phase 4  VERDICT    Write <project>/.reality/GAP-REPORT.md (template in
                    references/CLASSIFICATION.md). FIRST LINE is the greppable
                    verdict: `VERDICT: SHIP-READY` or `VERDICT: NOT SHIP-READY —
                    <top reason>`. SHIP-READY requires: every headline promise
                    WORKS via unmocked exercise · tests green · zero
                    CLAIMED-BUT-BROKEN anywhere. Anything less is NOT, with
                    blocking gaps ranked by intent impact.
Phase 5  INBOX      Judgment calls go to <project>/.autopilot/DECISIONS.md
                    (recommended option first), not into the verdict: drift that
                    might be deliberate ("code went Y, vision says X: which
                    wins?"), promises that maybe should be cut, UNDOCUMENTED
                    capability that maybe should be promoted to promise. The
                    skill reports; the operator arbitrates vision.
```

## EXTRACT Pipeline (when no intent source exists)

```
Phase E1  SIGNALS   Interview the codebase: entry points + what they wire up ·
                    what the tests assert the system must do · UI copy / CLI help
                    (the project pitching itself to users) · package metadata ·
                    commit-log story (what got built first, fixed hardest) ·
                    closed beads (revealed priorities: effort spent IS intent) ·
                    any project notes.
Phase E2  DRAFT     One-paragraph Intent Statement in the INTENT.md shape: who it
                    serves, what job it does, what "winning" looks like,
                    commercial vs fun. Plus 2–3 lines of "evidence for this
                    reading" and any signals that point a different direction.
Phase E3  CONFIRM   Write to <project>/.autopilot/INTENT.md headed
                    `STATUS: DRAFT — UNCONFIRMED (extracted by reality-check)`.
                    Surface to the operator (or the calling orchestrator, which confirms
                    with the operator). A reconstructed intent NEVER silently drives
                    destructive or directional moves. Confirmation flips the
                    header to CONFIRMED, and only then does ASSESS grade
                    against it.
```

## Contract (what orchestrators can rely on)

- `<project>/.reality/GAP-REPORT.md`: the gap report; first line is the
  greppable `VERDICT:` line. **ship-it** consumes this at Phase 0 PREFLIGHT as
  its go/no-go reality gate. **project-autopilot** consumes it as the assess
  step for vision-vs-reality drift (auto); each non-WORKS row is bead-ready.
- `<project>/.autopilot/INTENT.md`: EXTRACT mode's intent statement, written
  to the exact path project-autopilot already reads, DRAFT-flagged until the operator
  confirms. **idea-forge** and **project-autopilot** route here when a project
  lacks intent docs ("no intent = run reality-check's extraction first").
- `<project>/.autopilot/DECISIONS.md`: appended judgment items (the same inbox
  project-autopilot drains; one inbox, not two).
- Verdict echoed as the LAST line of the reply, so callers grep either place.
- Exit state: tree clean, nothing fixed, nothing committed. Reality-check ends
  where remediation begins.

## THE EXACT PROMPT

```
/reality-check
Target: <project path>
Mode:   assess | extract | auto     (default auto: assess, falling back to extract)
Focus:  <optional — one promise or area, e.g. "the booking flow" — narrows Phase 2>
```

## Anti-Patterns

- Grading from the README without running anything: the one failure mode this
  skill exists to prevent. No execution trace, no WORKS.
- Rounding PARTIAL up to WORKS to be encouraging. Flattery here costs a failed
  ship later; honest-and-specific IS the deliverable.
- Accepting a mocked test as proof of a headline promise. Mocks prove wiring,
  not delivery (shipped-reality standard).
- Fixing anything mid-assessment, even a one-line obvious bug. Report it,
  classify it, leave it. Read-only or the report is compromised.
- Skipping closed beads. They're the project's revealed priorities and the
  fastest drift detector (effort that went somewhere the vision never asked for).
- Treating an extracted DRAFT intent as confirmed, or grading against it before
  the operator signs off.
- Burying the verdict in prose. First line of the report, last line of the
  reply, greppable, every time.
- Launching the project bare on a unified-memory box (e.g. a DGX Spark) when
  memguard is available. Wrap anything heavy.
- Verdict-creep: SHIP-READY "except for…" is NOT SHIP-READY. The exception IS
  the reason.

## References

- [CLASSIFICATION.md](references/CLASSIFICATION.md): the six classes with
  evidence standards, the verdict rules, and the GAP-REPORT.md template
