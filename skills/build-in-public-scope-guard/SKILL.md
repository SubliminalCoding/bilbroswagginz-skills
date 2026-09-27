---
name: build-in-public-scope-guard
description: Triage rules for viewer suggestions during build-in-public streams (four bins, clearance bars, how to decline without being rude, common viewer patterns to recognize). Use when a stream viewer suggests a feature, framework swap, architectural change, or "have you tried X," when deciding whether to act on chat input mid-stream, or when reviewing post-stream chat logs to sort real signal from noise.
---

# Build in Public Scope Guard

A build-in-public stream has a structural problem: the audience is generating suggestions at a rate the builder cannot possibly sustain. Every suggestion feels low-cost to the person making it and high-cost to the person building. Without a triage system, the builder either becomes a suggestion-machine (everyone's ideas, nobody's product) or a suggestion-ignorer (which is rude and loses trust).

This skill is the triage system. It sorts every suggestion into one of four bins with explicit clearance bars, and it names the viewer patterns that are trying to exploit the stream's goodwill.

## The scarce resource

The scarce resource is the builder's time and focus, not the viewer's attention. This is not negotiable. Every suggestion taken has a cost: context switch, scope expansion, a distracted next thirty minutes. Every suggestion accepted is a negative on the builder's balance sheet until proven otherwise.

Viewer count does not change this math. One thousand viewers do not earn a thousand times more scope than ten. The cost of building a feature is the same whether one person suggested it or a hundred did.

## The four bins

Every suggestion goes into exactly one of these four bins. No "maybe." No "I'll think about it." Sort, act, move on.

1. **Implement on stream.** Rarely used. Only when the suggestion fits all three clearance bars below.
2. **Backlog.** Captured for post-stream review. Given a one-line note in the builder's log with the timestamp and the problem named.
3. **Decline with reason.** Acknowledged out loud on stream, with a one-sentence reason it doesn't fit. Not ignored.
4. **Ignore.** No acknowledgment. Reserved for suggestions matching the exploit patterns named below.

## Clearance bar for "implement on stream"

A suggestion goes into bin 1 only if **all three** of these are true. Not two of three. Not "close enough."

- **Fits the current scope.** The feature being built right now, not the adjacent one, not the one from last stream. If the builder is wiring up calendar integration and a viewer suggests adding SMS notifications, that's a scope miss even if SMS is a good idea.
- **Implementable in under thirty minutes.** Including the typing, the reading of chat reactions, the narration. If the suggestion is "add a CI pipeline," that's not a thirty-minute item and doesn't clear.
- **Has an acceptance test the audience can see.** Per `stream-demo-prep`. If the change can't be demonstrated on the current stream with a visible payoff, don't pick it up live. It becomes invisible work that looks like the stream stalled.

If any one of those fails, the suggestion moves to bin 2, 3, or 4.

## Clearance bar for "backlog"

Bin 2 is for suggestions that might become real work later. The bar is lower than bin 1 but still has teeth:

- **Names a real problem, not a preference.** "This feature would be useful for users who..." clears. "I'd personally prefer..." doesn't.
- **Not already on the backlog.** Don't add duplicates. If a similar item is already captured, note the corroboration on the existing entry and move on.
- **Not reversing an already-made decision.** A suggestion that would undo work already committed to (pick a framework, pick a deployment target, pick a data model) is bin 3, not bin 2, unless the viewer surfaces genuinely new information the builder didn't have.

## Clearance bar for "decline with reason"

Bin 3 is where most reasonable-but-wrong-for-now suggestions go. The bar is that declining is done **publicly** and **specifically**.

- **Publicly:** acknowledged on stream, so the viewer knows they were heard.
- **Specifically:** with a named reason that a thoughtful viewer can agree or disagree with. "Good idea, not today because scope" is the minimum viable decline; specific beats vague.
- **Without negotiation.** A decline is not an opening offer. Once declined, further advocacy by the viewer in that stream goes to bin 4.

Example decline: *"That's a real point about the caching layer, but it'd pull us off the current tenant-onboarding work, so it's going on the backlog for a dedicated pass later."*

## Clearance bar for "ignore"

Bin 4 is not laziness; it's a deliberate filter. These go into bin 4:

- **Framework shills.** "You should rewrite this in [language the viewer happens to use]." The viewer's preferences are not the builder's problem.
- **Tool recommendations with no stated problem.** "Have you tried [SaaS]?" without identifying what problem it would solve. If there's no problem named, there's nothing to evaluate.
- **Reverse-the-decision attempts on settled questions.** If the builder already picked Postgres and the viewer's suggestion is "use DynamoDB," the decision is not re-opened because a viewer showed up.
- **Concern trolling.** "Are you sure you want to do it that way?" without specifics. Respond only if the viewer names the specific concern; otherwise, bin 4.
- **Repetitive suggestions.** If the same viewer has made the same suggestion on three streams and it's been declined each time, further instances are bin 4.
- **Agenda comments.** Viewers promoting their own product, newsletter, or service under the guise of a suggestion. Bin 4.

The ignore bin is used without explanation. An explanation invites a continuation of a conversation that shouldn't start.

## Exploit patterns worth naming on stream

Some viewer patterns are worth calling out publicly once, not because the individual viewer deserves the callout, but because naming the pattern teaches the rest of the audience what the boundaries are.

- **The rabbit-hole question.** A question whose answer would take fifteen minutes, dropped mid-build. "Can you explain how your auth works?" is fine on a Q&A stream; during a build, it's "great question, saving it for the Q&A segment."
- **The feature-request-disguised-as-question.** "Do you support X?" where X doesn't exist is not a question; it's a feature request. Treat it like one.
- **The prestige demand.** "Real senior engineers would use [pattern]." Social pressure to reverse a decision. Name the pattern, then bin 4.
- **The concern escalation.** Starts as a small question, escalates to "I'm really worried about this architecturally" when the first nudge didn't land. Decline once with reason; don't re-engage.

Naming the pattern out loud ("I'm going to put that in the backlog rather than rabbit-hole it on air. Thanks for the question") is both a signal to the individual viewer and a training moment for the chat.

## Post-stream backlog triage

Bin 2 collects faster than it gets processed. Run a triage pass within 24 hours of each stream:

- **Real problem, fits the roadmap:** becomes a real task in the real tracker.
- **Real problem, doesn't fit:** archived with a one-line note on why.
- **On reflection, not actually a real problem:** deleted. Bin 2 was a judgment call in the moment; some of those calls were wrong, and that's fine.
- **Corroborates an existing task:** strengthens the case for that task. Note it.

Do not let bin 2 become a landfill. Items that sit in it for more than two streaming sessions without being promoted or archived are deleted on sight.

## When this skill should say no

- **Before implementing a viewer suggestion that doesn't clear all three bin-1 bars.** Stop. Send it to bin 2, 3, or 4.
- **Before ignoring a reasonable suggestion silently.** Stop. If it's reasonable, it's bin 3 (public decline with reason), not bin 4.
- **Before re-opening a settled decision because chat is pressuring.** Stop. The decision was made with more context than chat has. If new information arrives, reconsider; if it's just advocacy, bin 4.
- **Before going live on a build stream without this triage system already running in the builder's head.** Stop. The stream will generate suggestions. A builder without triage rules will either burn the session chasing them or lose the audience by ignoring them all.

## The underlying stance

Viewers' time is valuable. Their suggestions are often generous. None of that obligates the builder to build what they suggest. The deal of build-in-public is that the audience gets to watch the work happen, not that they get to direct it. Holding that line is what keeps the stream a stream and not a committee.
