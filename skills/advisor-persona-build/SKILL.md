---
name: advisor-persona-build
description: Meta-skill for building sellable advisor-persona skills (the /elon-hat, /myron pattern). Use when designing a new decision-making skill modeled on a specific real person's methodology, scoping which decisions a persona covers, sourcing corpus material, extracting the kernel of non-negotiable rules, or reviewing an existing persona skill for caricature vs. real signal.
---

# Advisor Persona Build

A good advisor persona is not a voice impression. It is a decision framework with attribution. The voice is the output format; the framework is the product. Skills that skip the framework and go straight to the voice produce caricatures that viewers can't tell apart from every other LinkedIn influencer.

This skill describes how to build one that's worth selling.

## The five ingredients

Every advisor-persona skill in this pack has the same five parts. If any one is missing, the persona is not ready to ship.

1. **The person.** Real, named, with a public methodology or documented decision history. Not "a wise founder"; Elon Musk. Not "a marketing legend"; Myron Golden. The person's name is the product name.
2. **The scope.** One decision domain. Hiring. Pricing. Roadmap cuts. Product positioning. Not "general business advice." A persona covering every domain is covering nothing; it becomes a chatbot in a costume.
3. **The corpus.** Quotes, transcripts, books, interviews, first-party recordings. Every rule in the kernel must trace to a specific passage. The corpus is the persona's receipts.
4. **The kernel.** Three to seven non-negotiable rules the person actually applies. Not inferred from vibes; extracted from the corpus with the source cited.
5. **The operator library.** Specific moves. "When [situation], [person] does [move]." Example: *when a hire's compensation demand exceeds the band, Musk either pays the premium and locks them in or walks that day. He doesn't negotiate in the middle.*

If the skill has all five, it's a product. If it has only #1 and #4, it's a vibe.

## How to scope the persona

Pick the decision domain first. Before touching the corpus, write one sentence:

*This skill is for [specific decision type] made by [specific person], and it overrides the default advice pattern by [one sentence describing the difference].*

If the sentence has to hedge ("generally," "in most cases"), the scope is too broad. Sharpen it until it's a claim.

Good scope examples:
- Elon Musk on roadmap cuts: apply the five-step algorithm (question every requirement, delete parts/processes, simplify, accelerate, automate, in that order) to any feature or process that's costing time.
- Myron Golden on high-ticket pricing: reframe the offer so the buyer sees the cost of not buying as larger than the price, using his specific problem-magnification patterns.

Bad scope examples:
- "Elon's advice on startups." Covers everything; commits to nothing.
- "Myron's sales wisdom." Same.

## Building the corpus

The corpus is the skill's defensibility. If your persona's rules all trace to verified quotes, the skill is auditable. If they don't, it's fan fiction.

Minimum corpus:
- **Primary sources.** Books the person wrote, direct interviews, talks they gave. Transcripts preferred.
- **Decision artifacts.** Documented cases where the person made the decision in question: company memos, earnings calls, case studies, tweet archives.
- **Contrasts.** Cases where the person did something surprising or against the expected pattern. These are often where the real kernel lives.

Quantity target: enough to extract 20+ direct quotes on the decision domain. If you can't find 20, the scope is too narrow or the person doesn't actually have a distinctive pattern in this domain.

## Extracting the kernel

The kernel is three to seven rules. Each rule has:

- **The rule itself:** one sentence, imperative.
- **The source:** specific quote or documented decision.
- **The test:** how you'd know if the rule applies to a given situation.

Example entry:

```
Rule: Question every requirement before accepting it, especially if attributed to "policy."
Source: Musk's five-step algorithm, Model Y production talk 2021.
Test: If a requirement's justification collapses to "that's how we've always done it" or "legal said so," the rule applies.
```

If a rule's source is "based on general reading of his work," delete the rule. Find a specific citation or drop it.

Cap at seven. More than seven means the persona has no spine; it's just a list of things. Force the cut.

## Building the operator library

The operator library is the "what does this person *do*" catalog. Each entry is a situation-to-move mapping, drawn from the corpus.

Format:

- **Situation:** concrete trigger.
- **The person's move:** what they actually do or have done.
- **Why:** the underlying reasoning, in the person's own framing.
- **Source:** corpus citation.

Target 10–20 operators per persona. Operators are where voice lives: the specific phrasing the person uses to justify the move becomes the template for the skill's output.

## Voice without caricature

Voice is the output format, not the product. Rules for voice:

- **Use the person's actual phrasing patterns**, sampled from the corpus. Don't invent phrases they "probably would say."
- **Avoid signature tics as decoration.** A persona that leads every response with "So, the thing is..." because the person sometimes does is a caricature. Use tics when they serve the content.
- **Keep profanity, opinions, and politics proportional to the corpus.** If the person curses once per hour on record, the skill doesn't curse in every response. If they dodge politics in interviews, the skill dodges politics.
- **Never attribute an opinion the person hasn't expressed.** If the skill is asked about a topic outside the corpus, it says so rather than improvising. "That's outside what [person] has addressed directly" is a valid output.

## Validators

A persona skill needs tests. Concrete ones:

- **Rule trace test.** For each rule in the kernel, the source citation should be retrievable. Broken citation = broken rule.
- **Caricature check.** Read five outputs. Would a person familiar with the advisor recognize the reasoning, or just the voice? If just the voice, the kernel is too thin.
- **Scope creep check.** Feed the skill a question outside the declared scope. If it answers confidently instead of declining or redirecting, the scope isn't being held.
- **Generic-influencer check.** Strip the name and voice tics from a response. If the remaining content could be from any business influencer, the persona isn't doing work.

Run these before shipping. Re-run them when you edit the kernel.

## When this skill should say no

- **Before starting a persona without a decision domain.** Force the scope sentence first.
- **Before extracting a kernel from a thin corpus.** If you can't find 20 direct quotes on the domain, either expand the corpus or pick a different persona.
- **Before shipping a persona that fails the generic-influencer check.** Something is missing from the kernel or operator library.
- **Before adding an eighth rule to the kernel.** Cut instead.
- **Before building a "general wisdom" persona.** No such product exists that's worth selling. Pick a domain.

## Why this is the force multiplier

Every persona shipped under this pattern is an instance of the same product line, not a one-off. The skill pack can grow to N personas without losing coherence, because each one was built the same way, with the same receipts, the same scope discipline, the same output shape. Viewers who buy one persona and like the framework will buy the next one. That is the pack's defensibility.
