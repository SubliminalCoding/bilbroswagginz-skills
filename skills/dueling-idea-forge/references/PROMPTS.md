# Duel Prompts

Send-ready templates for Phases 4–6. `<AGENT>` is the duelist receiving the prompt;
`<OPPONENT>` is the other seat. In a 3-way, send one cross-score prompt per opponent
and one reveal per scorer (6 artifacts each phase). Swarm form: deliver via
`ntm send`; gate the next phase on `ntm --robot-wait`, never on a timer.
Sequential form: each prompt IS the sub-agent/CLI invocation; the return is
the gate.

All prompts share two ground rules. Repeat them verbatim in every send:

```
GROUND RULES
- Judge against THIS project's reality (its docs, code, and existing br graph),
  not against generic best practice.
- Candor is the deliverable. Politeness that softens a real objection is a
  defect. So is manufactured disagreement to look tough.
```

## Phase 4: CROSS-SCORE

Send to both duelists at the same time. Neither has seen the other's file before this.

```
You are one of two different-lineage models that independently ideated on this
project. Below are your opponent's ideas. You have NOT seen them before; they have
not seen yours. Score every idea.

For EACH idea, output:

  IDEA <n>: <restate it in one line; proves you read it, not skimmed it>
  SCORE: <0–1000>
  RATIONALE: 3–6 sentences covering all four axes:
    1. QUALITY:         is the idea well-conceived, or plausible-sounding mush?
    2. USEFULNESS:      would a real user of THIS project feel the difference?
    3. PRACTICALITY:    can it actually be implemented in this codebase as it
                        exists? Name the file/subsystem it touches if you can.
    4. PAYOFF vs DEBT:  does the benefit justify the complexity and tech-debt
                        it introduces?
  STRONGEST POINT: the single best thing about this idea, even if you scored it low.
  WEAKEST POINT:   the single biggest flaw, even if you scored it high.

Calibration: 900+ = build this regardless of who proposed it; 700–899 = strong,
worth a bead; 400–699 = real but flawed; <400 = should not be built as stated.
Anchor scores to that scale. Do not grade on a curve or cluster everything at 700.

Write the full output to .ideaduel/scores-<AGENT>.md.

[GROUND RULES: paste block]

--- OPPONENT'S IDEAS ---
<contents of .ideaduel/ideas-<OPPONENT>.md>
```

## Phase 5: REVEAL

Send to each duelist its opponent's scores OF ITS OWN ideas.

```
Your opponent has scored your ideas. Their full scores and rationale are below.
React to EACH of your ideas with exactly one verdict:

  CONCEDE:    the critique is right. State plainly what you missed. (A clean
              concession is a win for the run, not a loss for you.)
  PUSH BACK:  the critique is wrong. Rebut the specific claim with specifics:
              point at the code, the user, or the constraint they misread.
              "I still think it's good" is not a rebuttal.
  REVISE:     the critique found a real flaw but the core survives. State the
              amended idea in one paragraph; it will be re-judged in this form.

Then close with two summary lines:
  CHANGED MY MIND ON: <idea numbers, or "none">
  STILL CONVINCED ON: <idea numbers where you push back hardest>

Defending all of your ideas is an automatic credibility hit: two unrelated
lineages never agree that completely. Write to .ideaduel/reveal-<AGENT>.md.

[GROUND RULES: paste block]

--- OPPONENT'S SCORES OF YOUR IDEAS ---
<the relevant section of .ideaduel/scores-<OPPONENT>.md>
```

## Phase 6a: REBUTTAL (optional)

```
Two targeted arguments, one paragraph of evidence each, no restating the reveal:

1. DEFEND your single most UNDERRATED idea (largest gap between your conviction
   and their score). Bring NEW evidence: a concrete user scenario, a pointer into
   the code, a cost estimate: something absent from your original write-up.
2. ATTACK your opponent's WEAKEST surviving idea (anything not yet conceded).
   Name the failure mode: what breaks, who gets hurt, what it costs later.

Append both to .ideaduel/reveal-<AGENT>.md under "## REBUTTAL".
```

## Phase 6b: STEELMAN (optional)

```
Write the strongest possible case FOR your opponent's #1-ranked idea. Argue it
as if it were yours and your reputation rode on it. Best concrete user story,
best implementation path in this codebase, best answer to the biggest objection
raised so far (including your own).

Then one closing line, honestly: "Steelmanned, I now score it <0–1000>."

Append to .ideaduel/reveal-<AGENT>.md under "## STEELMAN". (This round exists
because forced advocacy across lineage lines routinely surfaces value the
originator undersold and the scorer dismissed. Do it properly or not at all.)
```

## Phase 6c: BLIND-SPOT PROBE (optional)

Send to all duelists simultaneously.

```
You have now seen every idea from every seat in this duel. Final question:

What important, pragmatic, accretive idea for this project did NEITHER of you
produce? Not a variant or merge of an existing entry; something absent from
all files. Propose 1–3 candidates with the same one-paragraph reasoning depth
as your originals, and for each, one sentence on WHY it was missed (shared
framing? training bias? the docs steered you away?).

Append to .ideaduel/reveal-<AGENT>.md under "## BLIND-SPOT". Blind-spot ideas
both duelists independently surface go straight to the orchestrator as
consensus-winner candidates for Phase 7 scoring.
```

## Phase 7: score matrix shape (orchestrator, for DUEL-REPORT.md)

```
| # | Idea (one line) | Origin | Score by cc | Score by cod | [by gmi] | Reveal verdict | Bucket |
|---|-----------------|--------|------------|--------------|----------|----------------|--------|
```

Buckets: CONSENSUS WINNER (≥700 from every scorer) · CONTESTED (gap ≥300 or a
substantive push-back) · KILLED (low from all scorers AND conceded). Steelman
re-scores supersede Phase 4 scores for bucket math; note both in the row. Quote
each side's strongest argument verbatim under every CONTESTED row. Editorial
opinion goes ONLY in the final "## META-ANALYSIS" section.
