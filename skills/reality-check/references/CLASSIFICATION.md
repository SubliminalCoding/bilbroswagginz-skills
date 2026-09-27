# Classification rubric, evidence standards, verdict rules, report template

## The six classes

Grade each numbered promise into exactly one class. When torn between two,
take the worse one: this skill's bias runs opposite to the author's.

| Class | Meaning | Minimum evidence to assign it |
|---|---|---|
| **WORKS** | Promise delivered, end to end, as a user would hit it | An unmocked execution trace from THIS run: the command/flow you exercised + the output proving delivery. Tests alone never suffice for a headline promise. |
| **PARTIAL** | Core path delivers; promised edges/options/scale don't | Trace of what works + a specific named list of what doesn't (no vague "mostly works"). |
| **CLAIMED-BUT-BROKEN** | Docs assert it works; running it fails | The failing trace, verbatim. The most valuable class in the report: this is the gap nobody knows about. |
| **MISSING** | Promised, never built (or a stub/mock stands in for it) | Absence of implementation; grep for the entry point that should exist. A mock or hardcoded placeholder behind the promise = MISSING at shipped level, not PARTIAL. |
| **DRIFTED** | Effort went somewhere the vision never asked for, displacing or mutating the promise | The promise text vs what the code/beads actually built. Drift may be the right call; that's an inbox item, never the skill's call. |
| **UNDOCUMENTED** | Real, working capability no doc promises | Execution trace + absence from every intent source. Not a defect: surfaced so the operator can promote it to a promise or prune it. |

## Evidence standards (what counts, what doesn't)

- **Counts:** an end-to-end exercise from this run · a green test that hits real
  services (no mocks on the assertion path) · a reachable deployed URL you
  actually loaded · a fresh build that completed.
- **Supporting only (never sufficient alone for WORKS on a headline promise):**
  unit tests · mocked integration tests · CI badges · "it worked last week" ·
  the README saying so.
- **Stale evidence is no evidence.** A demo from a prior session doesn't carry;
  re-run it. Cheap to re-run, expensive to be wrong.
- Record the actual commands and trimmed output in the report. The report must
  let a skeptic re-run every claim.

## Verdict rules

```
SHIP-READY      ⟺  every headline promise = WORKS (unmocked)
                    AND test suite green
                    AND zero CLAIMED-BUT-BROKEN across ALL promises
NOT SHIP-READY  ⟺  anything less — name the top blocking gap in the verdict line
```

- Headline promises are the ones the project would demo first / a user pays
  for; flag them in Phase 0 so the verdict math is fixed before grading starts.
- PARTIAL on a headline promise → NOT. PARTIAL on a minor promise doesn't block
  by itself but is listed in blocking-adjacent gaps.
- No conditional verdicts. "SHIP-READY once X lands" = `NOT SHIP-READY — X`.

## GAP-REPORT.md template

```
VERDICT: SHIP-READY | NOT SHIP-READY — <one-line top reason>

# Reality check — <project> — <date>
Mode: assess          Intent source: <which doc(s)>          Focus: <if any>
Tracker: br read (N open / M closed) | unavailable

## Promise table
| # | Promise (headline ★) | Class | Evidence (cmd → result, one line) |
|---|---|---|---|

## Claimed-but-broken (the dangerous ones)
<per item: promise · what the docs say · the failing trace · likely blast radius>

## Drift narrative
<2–6 sentences: where effort actually went vs where the vision pointed,
and what that displaced. Omit section if no drift.>

## Top gaps, ranked by intent impact
1. <gap — why it matters to the stated intent — bead-ready one-liner>
2. ...

## Undocumented capability
<built-but-never-promised list, or "none">

## Inbox items written
<count + one-line each, mirrored in .autopilot/DECISIONS.md>

## Evidence appendix
<the commands run + trimmed output traces backing every WORKS and every
CLAIMED-BUT-BROKEN>
```
