---
name: overnight-docs-sweep
goal: Documentation always reflects what's actually in HEAD
cadence: nightly
run_via: /schedule (cloud cron) or local cron
routes_to: [readme-real, unslop]
stop_condition: A full diff of docs-vs-code surfaces no stale, missing, or AI-slop sections
verification_gate: Docs build/lint passes and the code green ratchet is untouched
---

# Overnight Docs Sweep

**Goal.** Wake up to docs that match the code. Catch drift introduced by the day's
commits before it compounds.

**Each iteration.**
1. Diff today's merged commits against the docs surface (README, `/docs`, API refs, CLI `--help`, env tables).
2. For each drifted area: update it to match the code, then run `unslop` on the rewrite so it doesn't read as AI filler.
3. Stage the change. Re-run the docs build/link-check.

**Stop when** a fresh pass finds no stale section, no undocumented public surface, and no slop. Open one PR with the batched fixes (never push straight to main).

**Green gate (every iteration).** Docs build + link check pass; the code green ratchet (`build + lint + tests`) is unchanged; this loop touches docs only.

## Copy-paste prompt
```
/loop Keep this repo's documentation in sync with HEAD. Each pass: diff the most
recent merged commits against README, /docs, API references, and CLI --help; update
any drifted or missing docs to match the code; run unslop on each rewrite so it
doesn't read as AI slop. Verify the docs build and link check pass and that the code
build/lint/tests are untouched. Stop when a full docs-vs-code pass surfaces nothing
stale, missing, or sloppy, then open a single PR with the batched changes. Do not
push to main.
```

**Cadence / notes.** Best as a nightly `/schedule` cloud agent. Pairs with [nightly-changelog](nightly-changelog.md) and `readme-real` for the README surface.
