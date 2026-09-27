---
name: dueling-idea-forge
description: >-
  Multi-model adversarial ideation: two different-lineage models (Fable + Codex by
  default) each run idea-forge independently, then score each other's ideas 0–1000,
  see how they were scored, and fight it out. The orchestrator synthesizes consensus
  winners / contested / killed, and consensus winners become br beads via idea-forge.
  Use when "dueling ideas", "adversarial brainstorm", "cross-model idea duel",
  "have two models argue about what to build", or when single-model ideation keeps
  producing plausible ideas you don't quite trust.
---

# Dueling Idea Forge

idea-forge with an opponent. Two models from *different training lineages* each
independently ideate on the same project, then judge each other's work blind, then
see the judgments and react. One model's top-5 is a hypothesis; two unrelated models
independently scoring the same idea 850+ is evidence. The duel doesn't generate
better ideas; it generates **trustworthy verdicts** about the ideas.

> **Core Insight:** Lineage diversity, not raw power, is what makes a duel worth
> running. The signal is cross-model DISAGREEMENT, and agreement *despite* different
> training biases. A strong model arguing against a different-lineage model beats a
> stronger model arguing against itself: same lineage on both sides is grading your
> own homework, and shared blind spots sail through unanimously. This is also why a
> "weaker" model can earn a seat if its lineage is genuinely different, and why a
> weak model from anywhere cannot (it injects noise, not a second perspective).

> **Core Insight #2:** The REVEAL is the highest-signal phase, not the scoring.
> Numbers are cheap; what a model does when confronted with an opposing score is
> not. A duelist that *concedes* its own idea after reading the critique has killed
> it more credibly than any rubric could. A duelist that *pushes back with specifics*
> has flagged a genuinely contested call that belongs in front of the user.

## Roster (fixed: do not improvise)

- **Default duel: Fable (Claude) vs Codex.** Two strong models, two lineages.
  - Claude duelist → NTM `cc` pane (swarm form) or an isolated Claude sub-agent
    (sequential form; see Preconditions).
  - Codex duelist → NTM `cod` pane, or a direct `codex` CLI invocation in the
    sequential form. Either way it is **pinned to the newest Codex model**
    (`--cod=1:gpt-5.5` or newer). Standing rule: codex-cli silently falls back to
    an older model unless pinned. After spawn, READ the model label in the pane
    and confirm before sending anything. A duel against last year's Codex is a
    weaker duel than you think you're running.
- **Gemini is the future third lineage.** When a `gmi` agent is available, run a
  3-way: every duelist scores both others (6 scoring artifacts, 6 reveals), and
  "consensus" means high marks from ALL scorers. Until then, 2-way is a legitimate
  duel; don't pad the roster to fake a third seat.
- **Never seat Qwen.** Standing rule: a small local model is not strong enough to
  judge or be judged; a weak duelist's scores are noise that pollutes the consensus math. There is
  deliberately no qwen- variant of this skill.
- **One lineage available → ABORT.** Say so plainly: "Only one model lineage is
  reachable; a model can't duel itself. Run /idea-forge solo instead." Never
  substitute two panes of the same lineage and call it a duel.

## Preconditions

1. Everything idea-forge requires (real project with intent docs, `br` initialized,
   late-stage guard: >10 modified files or >3 in-flight beads → ask the user first).
2. At least two lineages reachable (see Roster). `ntm` is OPTIONAL. With it the
   duel runs as parallel swarm panes; without it, run each duelist as a fresh,
   isolated sub-agent or CLI invocation (Agent-tool sub-agent for the Claude
   seat, a `codex` CLI call for the Codex seat), one phase at a time. A missing
   `ntm` never aborts a duel; only a missing second lineage does.
3. Budget awareness: a duel is ~2× idea-forge plus 3–5 exchange rounds. Use it when
   verdict confidence matters (direction-setting backlogs, "what's actually worth
   building" calls), not for routine backlog top-ups.

## The Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  DETECT     Confirm ≥2 lineages: cc reachable, cod spawnable on the newest
                    model, gmi if present. Pick the form: ntm swarm if installed,
                    else sequential isolated invocations. <2 lineages → ABORT
                    (see Roster).
Phase 1  SPAWN      Swarm form: ntm spawn <project> --cc=1 --cod=1:gpt-5.5
                    --no-user --stagger-mode=smart  (add --gmi=1 for a 3-way).
                    Verify the Codex pane's model label before proceeding.
                    Sequential form: no spawn; each later phase invokes the
                    duelists as fresh, isolated sub-agent/CLI calls.
Phase 2  STUDY      Both duelists read the project: AGENTS.md / README / CLAUDE.md
                    + the code's shape + existing br graph (open AND closed). Same
                    grounding idea-forge Phase 0 demands; ungroundedness disqualifies.
Phase 3  IDEATE     Each duelist runs idea-forge's diverge→converge (generate ~30,
                    winnow to a ranked top 5 with full rationale; expand toward 15
                    if asked) and writes ONLY to its own file:
                    .ideaduel/ideas-<agent>.md. ISOLATION IS THE POINT: neither
                    sees the other's ideas yet. No beads created in this phase.
Phase 4  CROSS-SCORE  The duel. Send each duelist the OTHER's ideas file,
                    simultaneously in the swarm form; in the sequential form,
                    strict context isolation does the same job (never paste one
                    scorer's output into the other's prompt). Either way neither
                    scores with knowledge of how it was scored. Every idea gets
                    0–1000 + candid written rationale on
                    four axes: quality, real-world usefulness, implementation
                    practicality here, payoff vs complexity/tech-debt. Output:
                    .ideaduel/scores-<scorer>.md. (Prompt: references/PROMPTS.md.)
Phase 5  REVEAL     Show each duelist how its OWN ideas were scored. It must react
                    per idea: CONCEDE (critique is right; say what it missed),
                    PUSH BACK (critique is wrong; say specifically why), or
                    REVISE (idea survives in amended form). Honest concession is
                    the deliverable; defending everything is an instant tell.
                    Output: .ideaduel/reveal-<agent>.md.
Phase 6  EXTRA ROUNDS (optional: run when the stakes justify the tokens)
                    REBUTTAL    defend your most underrated idea; attack the
                                opponent's weakest survivor.
                    STEELMAN    write the strongest possible case FOR the
                                opponent's #1. Forced advocacy across lineage
                                lines routinely surfaces value the originator
                                undersold and the scorer dismissed.
                    BLIND-SPOT  both answer: "what important idea did NEITHER
                                of us produce?" Adversarial pressure expands
                                both models' framing past their training bias.
Phase 7  SYNTHESIZE Orchestrator builds the score matrix and writes
                    .ideaduel/DUEL-REPORT.md with three buckets:
                    CONSENSUS WINNERS  high from ALL scorers (≥700 each as a
                                       default bar). The real signal: ideas that
                                       survived an unrelated lineage's bias.
                    CONTESTED          score gap ≥300, or originator pushed back
                                       with substance. Present BOTH sides'
                                       strongest argument, verbatim-faithful.
                                       Human judgment, not skill judgment.
                    KILLED             low from all scorers AND the originator
                                       conceded at reveal. Record the kill reason.
                    Report scores faithfully; editorializing lives ONLY in a
                    clearly-marked META-ANALYSIS section at the end.
Phase 8  OPERATIONALIZE
                    CONSENSUS WINNERS → idea-forge Phases 5–9 (dedup against open
                    beads, decision-inbox triage, br graph with self-contained
                    bodies + deps + test tasks, bv cycle-check, refinement passes).
                    Don't reimplement any of that here. Hand the winners list to
                    idea-forge and let it run its own gates.
                    CONTESTED → append to <project>/.autopilot/DECISIONS.md
                    (recommended side first, both arguments attached). Never
                    auto-build a contested idea.
                    KILLED → stay in the report as the audit trail. Done.
```

## Monitoring (light touch, swarm form only)

In the swarm form, duelists are panes; panes wedge. Between phases, glance at
`ntm --robot-tail` (or run your standard swarm-tending tick) before declaring a
phase stalled. A duelist mid-30-idea-diverge is slow, not stuck. Use
`ntm --robot-wait` to gate phase transitions rather than polling on a timer.
Don't build a cron for this. (Sequential form needs none of this; each
invocation returns when it's done.)

## Contract (what orchestrators can rely on)

- `.ideaduel/ideas-<agent>.md`: each duelist's independent ranked ideas (one per seat).
- `.ideaduel/scores-<scorer>.md`: 0–1000 + rationale for every opposing idea.
- `.ideaduel/reveal-<agent>.md`: concede / push back / revise, per idea.
- `.ideaduel/DUEL-REPORT.md`: score matrix + the three buckets + meta-analysis.
- **br**: consensus winners only, created via idea-forge (labeled `idea-forge`),
  `bv` clean.
- `.autopilot/DECISIONS.md`: contested items appended to the shared decision inbox.

## THE EXACT PROMPT

```
/dueling-idea-forge
Target:  <project path>
Focus:   <optional: dimension or area, passed through to both duelists' idea-forge runs>
Roster:  cc+cod (default) | cc+cod+gmi
Rounds:  core (default) | +rebuttal | +steelman | +blindspot | full
```

## Anti-Patterns

- Seating two panes of the same lineage and calling it a duel. That's one model
  grading its own homework with extra steps. Abort instead.
- Seating Qwen (or any weak model) "for a third opinion". A weak scorer's numbers
  corrupt the consensus math. Lineage diversity only counts between strong models.
- Spawning `cod` unpinned: you'll duel a silently-downgraded Codex. Pin newest,
  verify the pane label.
- Letting duelists see each other's ideas before Phase 4, or scores before Phase 5.
  Contaminated independence is the whole experiment voided.
- Letting the second scorer be influenced by the first. Send simultaneously in
  the swarm form; keep contexts strictly isolated in the sequential form.
- Averaging away disagreement. A 950-vs-300 idea is not a "625 idea", it's
  CONTESTED and it's the most interesting artifact in the run.
- Editorializing inside the score matrix. The orchestrator reports faithfully and
  opines only in the meta-analysis section.
- Auto-building contested ideas because one duelist was very persuasive. Contested
  means the user decides, full stop.
- Reimplementing diverge/winnow/bead-creation logic here. idea-forge owns all of
  it; this skill owns only the duel.
- Running a duel for a routine backlog top-up. That's solo /idea-forge; the duel
  is for verdicts that need to be trusted.

## References

- [PROMPTS.md](references/PROMPTS.md): exact send-ready templates for cross-score,
  reveal, rebuttal, steelman, and blind-spot probe.
- /idea-forge: the composed ideation + operationalization engine (Phases 3 and 8
  delegate to it wholesale).
- /ntm and /vibing-with-ntm, if installed: spawn syntax, robot-mode waits, swarm
  tending (swarm form only; without them, use the sequential form).
