---
name: advisor-council
description: Spawn multiple advisor-persona sub-agents (default Elon Musk, Naval Ravikant, Myron Golden) to debate the current project or decision competitively across multiple rounds, then synthesize where they agree, where they disagree, and the recommended call. Use when facing a meaningful decision about direction, scope, pricing, product shape, or strategy, and the value of hearing three well-reasoned competing takes exceeds the cost of the debate. Not for small or reversible decisions.
---

# Advisor Council

Three personas walk into the terminal. They read what you're working on, they read the memory, they read your notes, and then they fight (politely, in rounds) about what you should do. At the end there's a synthesis: where they agreed, where they didn't, and which direction wins on the merits.

This skill orchestrates that debate. It is not a novelty. It works because three opinionated frameworks pressure-test a decision better than one agreeable one, and because forcing the advisors to respond to each other prevents each of them from retreating into safe generic advice.

## Council composition

The council is assembled per-decision, not pre-built. Pick from the preset menu or compose a custom council.

**Preset menu (match to the decision type):**

- **Strategic-business-triad (default when unsure):** Elon-hat, Naval-hat, Myron-hat. Product scope + leverage/compounding + pricing/offer. Good for most monetization, positioning, and direction calls.
- **Product-launch-triad:** Elon-hat, Steve-Jobs-hat, Myron-hat. Ship-or-kill + product taste + offer construction. Good for go/no-go calls on a specific launch.
- **Hiring-triad:** Naval-hat, Ben-Horowitz-hat, Paul-Graham-hat. Specific knowledge + wartime leadership + founder pattern recognition. Good for key hires and firings.
- **Capital-allocation-triad:** Buffett-hat, Munger-hat, Naval-hat. Value + inversion + long-term game. Good for investment, M&A, and big-spend calls.
- **Positioning-triad:** Peter-Thiel-hat, Naval-hat, Myron-hat. Contrarianism + specific-knowledge + offer mechanics. Good for market-position and differentiation calls.

**Hat kernel brief:**

- **Elon-hat:** product scope, engineering tradeoffs, speed, deletion-first. Strongest on: should we build this at all, scope, what to cut.
- **Naval-hat:** leverage, specific knowledge, compounding bets, long-term games. Strongest on: is this the right game, does this compound, leverage shape of the deal.
- **Myron-hat:** pricing, offer construction, high-ticket positioning, buyer psychology. Strongest on: are we charging enough, is the offer legible, cost of not buying.
- **Jobs-hat** *(roadmap)*: product taste, ruthless no-saying, A-players only. Strongest on: is this product meaningfully better, what to cut for focus.
- **Bezos-hat** *(roadmap)*: long-term thinking, customer obsession, Day 1 mentality. Strongest on: strategic commitment, organizational design.
- **Buffett-hat / Munger-hat** *(roadmap)*: value, circle of competence, inversion. Strongest on: cost-of-being-wrong, capital allocation.
- **Graham-hat** *(roadmap)*: founder mode, make something people want, do things that don't scale. Strongest on: early-stage product decisions, founder psychology.
- **Thiel-hat** *(roadmap)*: contrarianism, monopoly theory, definite optimism. Strongest on: positioning, contrarian bets, market choice.
- **Horowitz-hat** *(roadmap)*: wartime/peacetime, hard things, hiring for strengths. Strongest on: leadership under pressure, cultural emergencies.

Hats marked *(roadmap)* are not yet built. Use `advisor-persona-build` to create them properly before including in a council; in the meantime, the council falls back to synthesizing a minimal persona from public corpus and flags the degraded kernel in the synthesis.

Custom councils override presets. For a specific decision that doesn't fit a preset cleanly, name the three (or five) hats most suited. A fourth hat who largely agrees with one already in the room is noise, not signal.

## Scaling the council

The council supports any odd number of personas from three to about nine. Tradeoffs:

- **Three:** the default. Fast (three parallel agents), easy to synthesize, forces crisp disagreement. Best for most decisions.
- **Five:** worth it when the decision has more than one kind of expertise at stake (e.g., product scope + pricing + technical risk + operational load + market timing). Synthesis takes longer; skip round 3 unless round 2 surfaced genuine splits.
- **Seven or nine:** rarely worth it. Diminishing returns: synthesis gets thin, agents start overlapping, wall-clock and cost grow linearly. Use only when the decision is genuinely five-dimensional and the user has the time to read the synthesis carefully.
- **Never even counts.** A four-advisor or six-advisor council produces tied splits on close calls, which forces the synthesis to choose arbitrarily or punt. Always odd.
- **Never two** (except the offline-persona fallback under "When this skill should say no"). A two-advisor debate is a coin flip with narration.

The ceiling is not "how many personas exist" but "how many add genuinely different frameworks to this specific decision." Adding a fourth persona who largely agrees with one of the three already in the room is noise, not signal.

Each hat must be backed by a persona built under the `advisor-persona-build` discipline. A hat without a real corpus, kernel, and operator library produces generic influencer output and wastes the council.

## When to convene the council

Running the council is not free. Three parallel sub-agents, multi-round debate, synthesis pass: the wall-clock and token cost is meaningful. Budget roughly **150-250K tokens across sub-agent outputs** for a three-persona, three-round council; the cost scales approximately linearly in both persona count and round count. Convene it when **all** of these are true:

- The decision is **meaningful**. Reversible, low-stakes calls don't need a council. Pick one and move on.
- The decision has **multiple defensible directions**. If the answer is obvious after thinking for thirty seconds, skip the council.
- The decision **benefits from competing frameworks**. Some questions are technical and have a correct answer found by measurement, not debate. Don't council those.
- You have **time to absorb the output**. Convening a council and then skimming the synthesis wastes everyone involved. If you can't spend fifteen minutes reading the output carefully, convene it later.

If any of those is false, this skill declines to run and suggests a cheaper path (read the memory, write the decision down, or pick one hat for a faster single-advisor pass).

## Pre-flight checklist

Before spawning round 1, verify:

1. **Every persona's SKILL.md exists and is readable.** Check each at `~/.claude/skills/<persona-name>/SKILL.md` (or the fallback paths in "Using existing persona skills"). If a persona is missing, do not proceed silently with a generic fallback. Flag it, name the degraded kernel in the synthesis, and ask the user whether to continue or swap in a different hat.
2. **The decision is phrased as a fork, not a topic.** "What should we do about pricing" is a topic; "Should we ship at $79 or $149" is a fork. The council produces useful output on forks, not topics. If the decision isn't fork-shaped, sharpen it before spawning.
3. **The persona count is odd.** See scaling section. Stop and recompose if even.
4. **The shared brief is under ~800 words and all four parts are present** (decision, project state, memory/stance, prior history). Brief bloat translates into sub-agent prompt bloat, which costs tokens for no gain.

## Context packaging

Each persona sub-agent needs the same shared context, packaged once and passed to all three. The shared brief has four parts:

1. **The decision.** One or two sentences stating the question. Not "what should I do about X," but "should I [specific action A] or [specific action B] by [concrete deadline]." Ambiguity here destroys the output.
2. **The project state.** The current working directory, the relevant files or commits, what has already been built, what constraints exist. Short. Dense.
3. **The operator's memory and stance.** Relevant entries from the agent's persistent memory and any standing-rules notes the operator keeps. The personas should argue within the operator's known constraints (e.g. deposit-first, no-spec-work), not against them.
4. **Prior history on this decision.** Any project notes or earlier conversations bearing on this call. Pull them with whatever search you have: ripgrep over the project's docs, or a knowledge-base search tool if one is set up.

The brief is assembled once by the orchestrator and passed identically to each sub-agent. The personas diverge because of their kernels, not because they received different facts.

## The debate structure

Three rounds, no more. Fourth-round debate is a sign the decision should be made, not further discussed.

**Round 1: Independent positions.** Spawn the three sub-agents in parallel, each with: the shared brief + their persona's kernel and operator library. Each returns a one-page position: their recommended direction, their top three reasons, their biggest objection to the decision as framed.

Parallelism matters. Running sequentially triples the wall clock and creates order-effects in the reasoning.

**Round 2: Reactions.** Each persona now sees the other two positions and is asked: where do you disagree with the others, what did they miss, what argument from the others would change your position, what wouldn't. This round is where the competition shows up. If round 2 produces bland agreement, the personas aren't doing their jobs and the kernels are too weak.

Inject only each persona's **final-answer block + top-three-reasons** from round 1 into the round 2 prompts, not the full round 1 output. Full outputs bloat round 2 prompts fast and don't add signal; the synthesis fields are what other personas need to react to. Keep round 2 prompts under ~800 words including the two competing summaries.

**Round 3: Final positions (conditional).** Only if at least one persona's **bottom-line recommendation shifted** in round 2, meaning a persona changed which option they're recommending, not just sharpened their reasoning for the same choice. If round 2 only hardened positions without movement, skip round 3; the debate is settled and more rounds are theater. If round 2 produced clean convergence on one option, also skip.

## Spawning mechanism

Two viable runtimes. Pick based on what's available.

**Claude sub-agents (preferred when available).** Use the Agent tool to spawn each persona in parallel. Each sub-agent gets a prompt composed of [persona kernel] + [operator library] + [shared brief] + [round instructions]. Prompts are the same shape across rounds; what changes is the round instructions and whether prior-round output is included.

Spawn three agents in a single message with parallel tool calls. Do not sequence them.

**Codex CLI (fallback, or when parallelism in the Claude session would be expensive).** Shell out to `codex` with the persona prompt as a file. Collect outputs, pass round-2 inputs in the same way.

**Choose Claude sub-agents when:** the orchestrating session has budget for three parallel agents and the personas are already defined as Claude skills.

**Choose Codex when:** the orchestrator wants to avoid spending its own context on three sub-agent outputs, or when the personas are defined as standalone prompt files more naturally invoked from a CLI.

Mixing is allowed (two Claude hats and one Codex hat), but it makes the parallel-spawn logic uglier, so do it only with a reason.

## Synthesis and output

After the debate ends, the orchestrator produces a synthesis. Four sections, in this order:

1. **Points of agreement.** Where all three personas, independently or after debate, converged. These are the strongest claims: three kernels arriving at the same answer is a real signal.
2. **Points of disagreement.** The specific moves the personas split on, with a one-sentence summary of each persona's position. No "both sides have a point" hedging. Name the split concretely.
3. **The recommended call.** The decision that best survives the debate. This is not always a majority-vote answer. Two tiebreaker rules for when round 2 doesn't converge:
   - **Load-bearing objection wins over majority.** If one persona's objection is load-bearing (the decision would fail if they're right on this specific point), the call goes their way even against the other two. Name the objection and why it outweighs the other two.
   - **Fastest-shipping path wins when no clear load-bearing objection exists.** When three distinct positions stand and none of them is obviously right, pick the one that ships fastest and embed the other two as watch-item tests. Fast shipping produces the evidence needed to falsify the losing positions; slow debate produces more debate.
4. **The watch items.** What would you need to see happen for the losing positions to be right? If those signals appear, revisit the decision.

The synthesis is shown to the user as the primary output. Full per-round transcripts are available on request but not the default view. Most of the value is in the synthesis, not the play-by-play.

## Using existing persona skills

If an advisor-persona skill is already installed (in this pack, in `~/.claude/skills/`, or in another agent runtime's skill directory you maintain), the council invokes it rather than reconstructing the persona inline. Check order:

1. `<this pack>/skills/<persona-name>/`
2. `~/.claude/skills/<persona-name>/`
3. Any other agent runtime's skill directory you've wired up (optional)

Found skill is used verbatim. Not found = synthesize a minimal persona from public corpus in the sub-agent prompt, flag in the synthesis that this persona is operating without its full kernel, and recommend building it properly with `advisor-persona-build` before the next council.

## Anti-patterns the council guards against

- **Agreeable output.** If all three personas agree on everything including framing, the kernels aren't strong enough or the decision was too obvious to need a council. Flag it and trust the obvious answer.
- **Ventriloquism.** Personas repeating surface tics of the real person without applying their decision framework. If Elon-hat just says "hardcore" and "first principles" without actually applying the five-step algorithm, the kernel is doing no work. Flag it.
- **Scope drift during debate.** Round 2 is for responding to the others, not for re-opening the decision question. If a persona wants to change the question being debated, the orchestrator rejects that move and holds the scope.
- **Fake consensus.** A synthesis that declares agreement where there was real disagreement is a lie. Name the split and let the user decide.

## When this skill should say no

- **Before convening the council for a reversible or low-stakes call.** Stop. Make the call and move on.
- **Before running with fewer than three personas by default.** A two-persona debate is often just a coin flip dressed up. Three is the minimum for the structure to work. Exception: one of the three is offline or undefined, and the remaining two have genuinely different kernels.
- **Before skipping round 2 to save time.** Round 2 is where the debate produces real signal. Cutting it turns the council into three parallel monologues.
- **Before accepting a synthesis that flattens real disagreement into false consensus.** Stop. Rewrite the synthesis to name the split.
- **Before running the council on a question with a measurable answer.** Measure it instead.

## Output shape the user should expect

A single synthesis document, readable in five minutes, containing the four sections above. Followed by an offer: "Full per-round transcript available on request."

That's the product: short and useful, not ceremony.
