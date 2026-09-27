# Routing: Ship It

Which installed skill does each phase (the non-pack ones only if installed; otherwise
do the step directly), the outward-button list, and the staged
prompts. ship-it adds the go/no-go gate, the button discipline, platform-decoupling,
and the post-ship smoke; these skills do the work.

## Phase → skill

| Phase | Skill |
|---|---|
| 0 PREFLIGHT: does it deliver? | reality-check; pre-ship hardening → /bug-zero, security-sweep; suite's green untrusted → /test-max; release carries a migration → /migrate-proof (contract steps are buttons) |
| 1 VERSION+NOTES: bump, notes, commit | release-preparations (version), changelog-md-workmanship, git-commit-craftsman |
| 2 BUILD: artifacts, checksums, installer | release-preparations (cross-platform build + checksums + GH release), gh-actions (CI), dsr / rch (optional fallback + builds, if installed), installer-workmanship (+ verify) |
| 3 STAGE: draft + preview, no publish | gh-cli (draft release), vercel / wrangler / gcloud (PREVIEW deploy), gh-og-share-images |
| 4 BUTTONS: outward, ask-user | (see button list) |
| 5 POST-SHIP: smoke + record | verify / run (smoke the released artifact/URL/binary) |

`release-preparations` is the spine (if installed): it already chains tests → version → build →
checksums → GH release → installer verify, with the dsr/rch fallback when GH Actions
is throttled (skip that rung if those helpers aren't installed; build locally or on
your own CI instead). Without it, run the same chain by hand. ship-it conducts it + the surrounding steps and enforces the
button rule.

## The outward buttons (NEVER auto; surface each with command + rollback)

| Button | Tool | Rollback |
|---|---|---|
| Push tag / release | git / gh-cli | delete remote tag (low-risk; the one the operator may pre-authorize) |
| Publish the GH release | gh-cli | un-publish / delete release |
| Promote preview → production | vercel / wrangler / gcloud | re-promote previous deployment |
| Submit to app store | (manual / eas) | new build; can't unsubmit; fully manual |
| cargo / npm publish | rust-crates-publishing / npm | yank (cannot truly delete); fully manual |

Truly irreversible (store submit, cargo/npm publish, prod deploy) stay manual or need
per-action confirmation even after a general go. A tag push is the only one the operator
typically pre-authorizes.

## Staged prompts

```
Phase 0: is this actually ready to ship? tests green? run reality-check: does the
         build deliver its intent? scan for secrets. give me a one-line GO / NO-GO
         recommendation. (the GO is mine.)

Phase 1: bump the version (semver), write the release notes / changelog, and make a
         clean conventional commit.

Phase 2: build the release artifacts (cross-platform via gh-actions; if actions is
         throttled, fall back to dsr/rch when installed, else a local build),
         generate checksums, build the installer and VERIFY it installs and runs.

Phase 3: draft the GH release (do NOT publish), draft the store metadata, deploy to a
         PREVIEW environment only, prep the share images.

Phase 4: show me the ready package and the exact publish buttons, each command and
         its rollback. press nothing. i'll tell you which to run.

Phase 5: after i ship, smoke the released thing (run the deployed URL / installed
         binary), record the version + rollback path, and tell me what's still in
         review per platform.
```
