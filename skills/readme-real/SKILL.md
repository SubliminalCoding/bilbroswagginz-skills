---
name: readme-real
description: Write or revise READMEs that sound like a human wrote them and that don't ship promises the code can't back. Bans AI-tell phrasing, hype words, "coming soon" sections, and feature claims without a repro command. Use when creating a README, rewriting one that reads as AI slop, deciding what features to list on a landing page, or reviewing someone else's doc for credibility.
---

# Readme Real

A README does two jobs: it tells a reader what the thing is and how to run it. Everything else (hype, personality, announcement energy, roadmap aspirations) either pays its way by advancing those two jobs or it gets cut. This skill holds the line.

The default output of most LLMs when asked to "write a README" violates this standard in predictable ways. This skill names those violations and forbids them.

## Banned phrasing

These constructions are forbidden in any README produced or reviewed under this skill. They do not get softer with context. They do not "sometimes fit." They get cut.

**The em-dash decoration pattern.** Em-dashes used as rhythmic decoration, the kind where every third sentence has one, inflate prose without adding structure. Use commas, periods, or parentheses instead. A README should have at most one or two em-dashes, total, and each should carry weight.

**The "it's not just X, it's Y" construction.** A dead giveaway of machine-generated prose. Every variant ("more than just a tool — a platform," "not merely a library, but a framework") is a dressed-up version of the same tell. Cut.

**Hype words without referents.** *Robust, powerful, seamless, world-class, best-in-class, blazingly fast, lightning-quick, cutting-edge, revolutionary, game-changing, next-generation, comprehensive, sophisticated, elegant.* These words do no work. If the thing is fast, show a number. If it's reliable, show the test suite or the uptime. Adjectives without evidence are decoration.

**Announcement energy.** "We're excited to announce." "We're thrilled to share." "Proud to present." A README is not a press release. Delete.

**AI-tell verbs.** *Delve, unlock, leverage, empower, boast, showcase, embark, navigate.* Each has a plain-English alternative that's shorter and truer. Use the alternative.

**"Simply" as a reassurance.** *"Simply run npm install."* If it's simple, it doesn't need the word. If it isn't simple, the word is a lie. Cut.

**Journey metaphors.** *"Your journey to X starts here."* *"Embark on your first project."* The reader is installing a library, not crossing a continent.

**The adjective stack.** *"A fast, reliable, scalable, type-safe, developer-friendly..."* More than two adjectives in a row means none of them are load-bearing. Pick the one that matters and cut the rest.

**Emoji headers.** Decorative emoji in section titles (✨ Features, 🚀 Getting Started, 📦 Installation) signal "a language model wrote this README." Headers should be words, structured by a table of contents if the README is long enough to need one.

**"Coming soon" sections.** Either it ships or it doesn't exist in the README. Roadmap aspirations belong in a separate `ROADMAP.md` or an issue tracker, not in the README where they read as promises.

## The repro rule

Every feature claim in a README must be followed by a way for the reader to verify it. "Feature claim" means any sentence that says the software does something.

Good:
```
Supports JSON output via `--json`:

    $ tool --json foo
    {"result": "bar"}
```

Bad:
```
Supports a variety of output formats for maximum flexibility.
```

The first one is falsifiable. The second is decoration.

This applies to install, usage, configuration, and features. If the claim can't be demonstrated with a shell command, a config snippet, or a screenshot showing the actual output, the claim doesn't belong in the README yet.

## Structure

A README has these sections, in this order. Sections not on this list require a reason to include.

1. **What it is.** One sentence. Reader knows whether to keep reading by the end of the first line.
2. **Install.** Commands that actually work on a fresh machine. If prereqs exist, name them or include a one-liner that installs them.
3. **Quick start.** Shortest path from install to a visible result. Commands plus the expected output.
4. **Usage or Features.** Each item has a repro per the rule above.
5. **Configuration.** Only if configurable. Defaults named. Each option has a reason to change it.
6. **Limitations.** Named, not hidden. What it doesn't do, what it doesn't scale to, what it won't fix for you.
7. **License.** One line if using a standard license. Link to `LICENSE` for the full text.
8. **Contributing.** Only if contributions are actually accepted. If they're not, leave this out rather than writing a dishonest "contributions welcome."

Skip the "About the author" section, the "Star history" embed, the "Sponsors" collage, the "Star us on GitHub" call to action. None of them help a reader decide whether to install the thing.

## Badges

Badges earn their place by affecting a reader's decision. Keep:

- **Build status.** Does the current code actually build?
- **Version.** What version does `install` fetch?
- **License.** Decision-relevant for legal/commercial use.

Remove:

- Star counts. Popularity is not a quality signal for this reader.
- Visit counters.
- "Made with [language]" badges. The install commands already tell them.
- Shields that link to the project's own contributors page.

Two or three badges maximum. A header strip of eight badges is decoration.

## Voice

Present tense. Active voice. Second person when addressing the reader directly ("you pass `--json` to enable..."). First person plural ("we," "our") is fine when describing what the project does as a collective action; it is not fine as a stand-in for hype ("we're excited to...", see above).

Short sentences. A README is read fast, often on mobile, often by someone deciding in twenty seconds whether to bookmark or close the tab. Long paragraphs signal that the author didn't cut.

No quotes from the team. No "what people are saying." No testimonials. If the project has users whose endorsement matters, they can write their own blog post linking to the project.

## Anti-checklist

Before a README ships, read it once looking for these and cut each one:

- Any sentence containing "robust," "powerful," "seamless," or any banned word above.
- Any claim that isn't followed by a repro.
- Any emoji in a header.
- Any "coming soon."
- Any em-dash used for emphasis rather than necessity.
- Any paragraph longer than four sentences.
- Any adjective stack of three or more.
- Any "we're excited" or equivalent.
- Any section the reader doesn't need to decide whether to install.

If the count of cut candidates is zero, the reviewer isn't looking hard enough. Look again.

## When this skill should say no

- **Before shipping a README that calls the project "robust," "powerful," or "world-class."** Stop. Either delete the word or replace it with evidence.
- **Before publishing a feature list where the features aren't demonstrated.** Stop. Add repro commands or cut the claims.
- **Before adding a "coming soon" section.** Stop. Ship it first or move it to a separate roadmap file.
- **Before accepting LLM-drafted README text unedited.** Stop. Run the anti-checklist. Default LLM output will fail most of the bans above on the first draft.
