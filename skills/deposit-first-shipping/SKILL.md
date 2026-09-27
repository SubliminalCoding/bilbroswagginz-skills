---
name: deposit-first-shipping
description: >-
  Release discipline for work that has to pay for itself: no shipping without a
  named customer, no 1.0 without cleared payment or a committed buyer, no polish
  passes on features nobody uses. Use when tagging a release, deciding whether a
  project is ready to demo publicly, choosing between "ship a rough v1" and "polish
  more," or evaluating whether to continue investing time in a build.
---

# Deposit-First Shipping

Releases are accounting events, not vanity events. This skill gates every ship against a simple question: who is this release for, and has that someone put anything on the table?

If the answer is "nobody yet, but I think people will want it," the release is not a release. It's a WIP branch. Keep working on it, but don't brand it, don't tag it, don't promote it.

## The ship test

Before tagging any release (client deliverable, product launch, streaming demo, anything with a version number attached), all three must be true:

1. **The release has a named recipient.** A specific client, a specific user cohort, or a specific product hypothesis with a specific test audience. "People on the internet" is not a recipient.
2. **That recipient has committed something.** A cleared deposit (client work), a signup with a card on file (SaaS), a committed demo slot (enterprise pilot), a streaming audience expecting delivery on a specific date. Intention does not count. Verbal agreement does not count. Something has to be on the table.
3. **The release has an acceptance test the recipient can run.** Not a changelog. An actual thing the recipient does that proves the release did what it promised. Call the demo line, place a test order, run the onboarding flow.

If any one of these fails, the work continues but does not ship as a release.

## Version numbers mean something

Version numbers are public promises. Treat them that way.

- **0.x:** Working on it. Not asking anyone to depend on it. Breaking changes without notice are allowed. No marketing.
- **1.0:** Someone has paid for this or committed to pay. A specific person can name what the software does and run the acceptance test. This number is a contract with that person.
- **1.x patch:** Bug fixes only. No new scope. If new scope shows up, it's a minor bump.
- **2.0:** A new 1.0 for a different or expanded contract. Same bar as the original 1.0.

Skipping straight to 1.0 because the README looks polished is a lie. The number tracks the commitment, not the quality of the copy.

## The "ship a rough v1" rule

For product/streaming work where there is no client deposit yet, the discipline flips but does not disappear:

- **Ship the end-to-end path working before polishing any segment of it.** A demo that takes the user from landing page to completed core action, even if ugly, beats a beautiful landing page with nothing behind it.
- **The demo tells the truth.** If the demo requires the operator to be in the loop manually, that is not shipped. If the demo only works on one phone number, it is a prototype, not a product.
- **Polish is paid for by usage, not by vibes.** If nobody has used the rough version, do not polish it. Get it in front of the named recipient first. The polish list writes itself from their actual friction.
- **Streaming counts as a recipient.** If the operator is going to demo this on stream, the stream audience is the named recipient, the stream date is the acceptance test, and "it works on air" is the ship test. That is a valid release gate.

## When this skill should say no

Stop and challenge the plan when:

- **A release is about to go out with no named recipient.** Push for a named recipient before shipping. "I'll figure out who wants it after" is how projects rot on the shelf.
- **Polish work is queued on a feature with no users.** The polish is premature. Go find a user first; the feature either has legs or doesn't.
- **A 1.0 is being tagged without a committed buyer or audience.** Suggest 0.x instead until the commitment exists.
- **A "just ship it" impulse is about to release something that fails the acceptance test.** The acceptance test exists because the operator's past self knew their present self would want to cut the corner. Don't cut it.
- **Scope is being added to a release that was supposed to close.** The release closes; the new scope goes into the next release. A release that keeps growing never ships.

## What a clean ship looks like

A ship is boring:

- Tag the version.
- Write three lines in the release notes: who it's for, what it does, how they verify it works.
- Send it to the named recipient.
- Watch them run the acceptance test.
- Move on to the next thing.

No grand launch post, no "excited to announce," no polish pass inspired by how the release notes look. The release is the thing. The marketing, if any, is a separate task with its own scope and its own deadline.

## The reverse question

Any time the urge shows up to keep polishing before shipping, ask:

*Who specifically is waiting for the polished version, and what have they put on the table for it?*

If the answer is vague, ship the rough version now. If the answer is specific, finish the polish and ship to that specific person. Either way, the answer ends with shipping. The one path this skill forbids is the third option: polish forever for an audience that might exist someday.
