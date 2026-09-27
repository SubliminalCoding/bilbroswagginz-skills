# Rubric: Idea Forge

How candidates get generated (Phase 1) and judged (Phase 2). The dimensions seed
diversity on the way out and score quality on the way back in.

## The ten dimensions

| Dimension | The question it asks of a candidate |
|---|---|
| **Robust** | Does it survive bad input, partial failure, weird state? Fewer ways to break. |
| **Reliable** | Does it make behavior consistent and recoverable? Same input, same outcome, every time. |
| **Performant** | Does a user-felt latency or throughput number actually move? (Not micro-opts nobody perceives.) |
| **Intuitive** | Does the next action become obvious without docs? Less to explain. |
| **User-friendly** | Does it reduce friction, error messages decoded, steps removed for the human? |
| **Ergonomic** | Does it fit the operator's hands: CLI flags, defaults, hot paths shortened for the daily user? |
| **Useful** | Does it do a job someone demonstrably has, today, in this project? |
| **Compelling** | Would a user mention it unprompted? The demo-able, wow-adjacent quality. |
| **Accretive** | Does it ADD onto what exists without breaking or rewriting it? (Gate, not score: fails this, it's out.) |
| **Pragmatic** | Buildable in days with the current stack and skills? (Gate, not score: moonshots are out.) |

Accretive and pragmatic are **gates**: a candidate failing either is discarded in
Phase 1, no matter how well it scores elsewhere. The other eight are **scores**.

## Phase 1: using the dimensions to diverge

Sweep dimension by dimension: "what would make this more robust? ... more
compelling?" 3–5 candidates each gets you to ~30 with built-in diversity.
A candidate may serve several dimensions; tag it with its primary one. If a Focus
was given in the prompt, weight that dimension's sweep heavier, don't skip the rest.

## Phase 2: the four convergence questions

Every candidate gets all four, in writing, before any ranking:

1. **How does it work?** Mechanism, in this codebase's terms. If you can't
   sketch the mechanism, you don't have an idea, you have a wish.
2. **How does a user perceive it?** What changes in their session, concretely.
   "Cleaner code" perceives as nothing; nothing scores zero here.
3. **How is it implemented here?** Which files/modules move, what it touches,
   what it risks. Grounded in the Phase 0 read, not a hypothetical repo.
4. **Complexity vs payoff?** Honest size (S/M/L) against honest impact
   (low/med/high). L-complexity/low-impact dies here regardless of how clever it is.

## Ranking

Score the eight non-gate dimensions 0–2 (absent / present / strong) per candidate.
The score ORDERS the deep look; it never replaces it. Final rank comes from the
four questions: a candidate with a crisp mechanism, a perceivable user change, a
contained implementation, and S-or-M complexity for med-or-high impact beats a
higher raw score that's fuzzy on question 1 or 3. Ties break toward the dimension
the project is currently weakest in (per the Phase 0 ground).

## Ranks 6–15 (Phase 4)

Don't re-generate; pull them from the same scored pool. Prefer candidates that
*complement* the top 5 over near-duplicates of them: the observability for #2's
feature, the docs for #1's API, the test harness #4 will need. A weaker idea that
multiplies a winner outranks a stronger idea that overlaps one.
