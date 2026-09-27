---
name: unslop
description: >-
  Line-level copy editor that strips the tells of AI-generated writing (filler
  openers, hype adjectives, empty tricolons, hedges, em-dash rhythm, false
  balance, restating conclusions, emoji-bullet decoration) while a meaning
  guard keeps every claim, number, and instruction intact. Covers docs AND the
  product surface (buttons, toasts, errors, empty states, onboarding), and can
  edit toward a specific voice instead of generic-neutral. Use when "this reads
  like AI", "de-slop this", "fix this copy", "make this sound human", "clean up
  the UI text", or when ux-scan / ux-maximizer / project-autopilot surface
  AI-slop tells in docs or UI copy.
---

# Unslop

Point it at text (a doc, a React component, a pasted draft) and it returns
the same content written the way a sharp human would have written it, plus a
change log of every edit and why. It is a style edit with a hard floor: meaning
survives untouched. What gets cut is the residue models leave behind when
nobody edits them: the throat-clearing opener, the adjective that proves
nothing, the third list item that exists for rhythm, the conclusion that
restates the page.

> **Core Insight:** Slop is not a vocabulary problem, so a synonym pass can't
> fix it. The tells live as much in *distribution* (uniform sentence length,
> an em-dash every third line, every bullet bold-led) as in "delve" and
> "seamless." Two consequences drive the pipeline: the meaning guard runs
> **before** the edit, because the cheapest way to make a sentence flow is to
> quietly weaken its claim; and the output gets **re-scanned**, because the
> same instincts that produced the slop will re-introduce it mid-rewrite.

> **Core Insight #2:** The highest-damage slop isn't in the README. It's in
> the product. Users read every button label and every error message; almost
> nobody reads the docs. "Oops! Something went wrong 🎉"-grade copy on an
> error toast costs trust at the exact moment the product is already failing.
> So ui-copy is a first-class mode here, not an afterthought.

## Preconditions

1. **A concrete target**: a file path, a component, or pasted text. Unslop
   edits; it doesn't hunt. For a whole-project sweep, get a target list first
   (ux-scan surfaces copy findings; project-autopilot routes them here one at
   a time).
2. **A resolvable mode**: infer when obvious: `.md`/`.mdx` → docs;
   `.tsx`/`.jsx`/locale JSON → ui-copy; "my tweet/post/email/landing copy" →
   personal-voice. Ambiguous → ask, don't guess.
3. **personal-voice mode requires a voice profile loaded**: read your
   voice-profile skill (e.g. `personal-voice`) if one is installed, before touching
   a word. Its rules override this skill's defaults wherever they conflict
   (e.g. a total em-dash ban beats "at most one or two"). No voice-profile
   skill installed → say so and fall back to clean-neutral (docs-mode) editing.
4. **File targets should be diffable** (in git, or back up first); the
   style-only contract is verified by reading the diff.

## The Pipeline (staged: each phase finishes before the next fires)

```
Phase 0  SCOPE    Resolve mode + voice target. docs → clean, plain, evidence-
                  first. ui-copy → terse, on-brand, action-first (take a Voice
                  note if the brand has one). personal-voice → load the voice
                  profile, edit TOWARD that voice, never toward generic-neutral.
Phase 1  EXTRACT  Pull only the strings a human reads. Docs: prose blocks,
                  never code fences, commands, or output samples. UI: JSX text
                  nodes, button/label/placeholder/aria-label/title props,
                  toast() calls, error-message constants, empty-state copy,
                  Zod messages, metadata descriptions, locale-file VALUES
                  (never keys). Everything else is off-limits.
Phase 2  MARK     Scan extracted strings against the tell taxonomy
                  (references/TELLS.md). Tag every hit with its class ID
                  (T1–T9 prose, U1–U5 UI). Include the distribution tells:
                  count em-dashes, measure sentence-length variance, check
                  whether every bullet opens bold. No tag, no edit.
Phase 3  GUARD    Meaning audit, NON-SKIPPABLE. For each marked span: would
                  the natural fix change a technical claim, a number, a
                  warning, an instruction, or an API/product name? If yes →
                  do NOT edit; move it to FLAGS with the proposed rewrite and
                  what it would alter. The human decides; the skill doesn't.
Phase 4  EDIT     Rewrite the surviving spans toward the voice target. This is
                  structural, not lexical: cut the filler opener entirely
                  (don't replace it), collapse the tricolon to the one item
                  that's true, delete the hedge or commit to the claim, break
                  the uniform rhythm by varying sentence length, let the
                  conclusion end instead of recap. De-slopped text is almost
                  always SHORTER; growth is a defect.
Phase 5  VERIFY   Re-run Phase 2 on the output (rewrites breed new tells).
                  Diff check: zero changes outside extracted strings: no
                  code, identifiers, keys, numbers, or claims moved. UI mode:
                  length budgets hold (buttons ≤ 3 words, toasts ≤ 8, no
                  layout-breaking growth). Any miss → one more EDIT pass on
                  the misses only, then stop.
```

## Mode rules

**docs**: voice is clean and direct: present tense, active, short sentences,
claims backed or cut. This mode edits sentences inside an existing structure.
If the README's *structure* is the problem (sections, badges, repro gaps),
that's readme-real's job. Hand off rather than restructure here.

**ui-copy**: every string answers one question. Buttons say the action
("Upgrade", not "Unlock Premium Now!"). Errors say what happened and what to
do next ("Couldn't save. Check your connection and retry", not "Oops!
Something went wrong 😞"). Empty states orient ("No invoices yet. Create your
first one above."), they don't sell. Toasts confirm in ≤ 8 words ("Saved",
not "Awesome! Your changes have been successfully saved! 🎉"). Onboarding
instructs; nobody is on a journey.

**personal-voice**: the target is a specific person's voice, not neutral-human.
Load whatever voice-profile skill is installed (e.g. `personal-voice`); its
mandatory rules apply on top of the taxonomy: things like opinion attached to
facts, no em-dashes at all, no hashtags, no "here's the thing." A de-slop that
lands on bland is a failure in this mode even if every tell is gone.

## Contract (what orchestrators can rely on)

ux-maximizer lists this skill for UI copy, ux-scan pairs its copy findings to
it, and project-autopilot routes "AI-slop tells in docs/UI copy" here. The
interface is fixed:

- **Input:** a target (file path, component, or pasted text) + a mode
  (`docs | ui-copy | personal-voice`).
- **Output, file target:** edits applied in place, strings only; `git diff`
  shows no logic, identifier, or key changes. Pasted text: the cleaned text
  returned in full.
- **CHANGES log, always returned in the reply:** one line per edit:
  `<class-ID> | <before> → <after> | <why>`.
- **FLAGS, always returned (may be empty):** spans the meaning guard refused
  to touch, each with the proposed rewrite and exactly what claim it would
  alter. Flagged spans ship unedited.
- **Style-only guarantee:** behavior, claims, numbers, and instructions are
  identical before and after. An orchestrator can apply unslop output without
  re-verifying facts.

## THE EXACT PROMPT

```
/unslop
Target: <file path | component path | "text:" followed by the pasted text>
Mode:   <docs | ui-copy | personal-voice>   (optional; inferred when obvious)
Voice:  <optional brand notes for third-party copy, e.g. "clinical, calm">
```

## Anti-Patterns

- Editing content to fix style: changing a number, weakening a claim, or
  dropping a warning because the sentence reads better without it. That's
  what the GUARD exists to catch; anything it flags ships unedited.
- Synonym-pass editing: swapping "robust" for "solid" and calling the
  sentence fixed. The structural tells (rhythm, tricolons, restating closes)
  survive every thesaurus.
- Editing voiced content toward generic-neutral. personal-voice mode exists
  because "no tells" and "sounds like the author" are different targets.
- Touching anything a compiler or test reads: code, identifiers, i18n keys,
  CLI flags, config values. Strings a human reads, nothing else.
- Skipping the re-scan because the rewrite "reads clean". First-pass rewrites
  reintroduce tells at a rate that makes Phase 5 non-optional.
- Restructuring a README: section order, badges, and repro rules belong to
  readme-real. Unslop is the line-level editor inside whatever structure
  already exists.
- Letting the copy grow. De-slopped text shortens. If the diff adds words,
  something got explained that used to be decorated, and it should be checked.
- Sanding off deliberate personality because it pattern-matches a tell. An
  intentional brand catchphrase is voice, not slop. When in doubt, flag it.

## References

- [TELLS.md](references/TELLS.md): the full tell taxonomy: T1–T9 prose
  tells and U1–U5 UI tells, each with detection cues and before/after pairs
