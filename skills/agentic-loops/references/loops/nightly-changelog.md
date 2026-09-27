---
name: nightly-changelog
goal: Keep CHANGELOG tracking what actually shipped, in user-relevant language
cadence: nightly
run_via: /schedule (nightly)
routes_to: [unslop, ship-it]
stop_condition: Every user-relevant change since the last entry is recorded; CHANGELOG's unreleased section matches the day's merged commits
verification_gate: Each entry traces to a real commit/PR/bead AND reads as a user-facing change (not a raw commit subject); docs build passes
---

# Nightly Changelog

**Goal.** A changelog that's actually current is the source feed for release notes
and build-in-public posts. This keeps it honest nightly so you never reconstruct a
month of history at release time.

**Each iteration.**
1. Diff the day's merged commits/PRs/closed beads against the CHANGELOG's unreleased section.
2. For each user-relevant change, add an entry in user-facing language (not the raw commit subject); run `unslop` on it so it reads human.
3. Drop pure-internal churn (refactors, lint) unless it changes behavior.

**Stop when** every user-relevant change since the last entry is recorded and the unreleased section matches the day's merged work.

**Verification gate.** Each entry traces to a real commit/PR/bead AND reads as a user-facing change; a raw commit subject pasted in is FAILED. Docs/markdown build passes.

## Copy-paste prompt
```
/loop Keep CHANGELOG current nightly. Each pass: diff today's merged commits/PRs/
closed beads against the unreleased section; add an entry in user-facing language for
each user-relevant change (run unslop so it reads human); skip pure-internal churn
unless behavior changed. Each entry must trace to a real commit/PR/bead and read as a
user-facing change, not a raw commit subject. Stop when every user-relevant change
since the last entry is recorded.
```

**Cadence / notes.** Nightly via `/schedule`. Feeds [content-from-shipped-work](content-from-shipped-work.md) and your release prep.
