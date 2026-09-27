---
name: ship-it
description: >-
  Take a green project to a released, deployed artifact by orchestrating the
  release skills you have installed to do ALL the reversible pre-flight perfectly (preflight
  gate, version bump, changelog, cross-platform build, checksums, installer verify,
  preview deploy, drafted release + store metadata) and then handing you the exact
  publish buttons. Use when "ship it", "cut a release", "prepare to launch",
  "tag a release", "deploy to production", "submit to the store", "publish".
---

# Ship It

The last-mile vertical. The quality siblings (bug-zero / perf-max / ux-maximizer)
get a project to *good*; ship-it gets it *out the door*. It conducts your release
skills to do every reversible step flawlessly, then stops at each outward button and
hands it to the operator. Shipping is irreversible and outward-facing, so the button
is always the operator's.

> **Core Insight:** ship-it is the *inverse* of the quality verticals. They're
> mostly-auto with a few judgment calls; this is mostly *ask-user*, because publish /
> prod-deploy / store-submit / cargo-publish / push are forbidden-to-auto by the
> rails. Its value isn't pressing buttons; it's making the button a 30-second
> decision instead of an hour of prep. Everything reversible (version, changelog,
> build, checksums, preview deploy, drafted release) is auto; everything outward is a
> staged button with a rollback path. Like its siblings it has a **green ratchet** and
> a **decision inbox**; here the inbox is mostly "here are the buttons."

## Preconditions / guards (the go/no-go gate)

1. **Green + actually delivers, and the green is TRUSTED.** Tests green AND a
   reality-check says the build delivers its intent (don't ship broken, and don't
   ship something that doesn't do what it claims). If pre-ship hardening is
   warranted, run **/bug-zero** and/or security-sweep FIRST. And if the suite's
   green can't be trusted (weak assertions, flakes, line-coverage theater), run
   **/test-max** before its green counts. This gate is only as strong as the tests
   behind it.
2. **No secrets in the release.** Scan artifacts/config; never ship a key.
3. **Clean tree + rollback path named** before any outward step.
4. **A release that includes a migration ships under /migrate-proof.** Its
   expand → migrate → contract gate wraps the release: expand steps may ride this
   one; contract (killing the old path) waits for the parity oracle + a rehearsed
   rollback, and every contract step is a button.
5. **The go is the operator's.** ship-it prepares and recommends; it never decides to ship.

## The Pipeline (staged: reversible work auto; outward steps = buttons)

```
Phase 0  PREFLIGHT   Go/no-go gate: tests green AND trusted (test-max if not) ·
                     reality-check delivers · pre-ship hardening done if warranted ·
                     any migration gated by migrate-proof · no secrets · clean tree. Output a
                     one-line go/no-go recommendation (the GO is the operator's).            [auto check]
Phase 1  VERSION+NOTES  Bump version (semver), write release notes / changelog
                     (changelog-md-workmanship), clean conventional commit
                     (git-commit-craftsman).                                          [auto, reversible]
Phase 2  BUILD       Build release artifacts: cross-platform binaries via gh-actions
                     CI (fallback: dsr/rch if installed, else your normal CI or a
                     local build); checksums; installer (installer-workmanship)
                     and VERIFY the installer actually works.                          [auto, reversible]
Phase 3  STAGE       Everything that surrounds the release but stops short of publish:
                     DRAFT the GH release (unpublished), draft store metadata, deploy
                     to a PREVIEW env (vercel preview / staging, never prod), prep
                     OG/share images.                                                 [auto, reversible]
Phase 4  THE BUTTONS Present the operator the ready package with the EXACT publish actions and
                     each one's rollback path: push the tag · publish the GH release ·
                     promote preview→production · submit to the store · cargo publish.
                     ship-it does NOT press these. On the operator's explicit per-action go, it
                     may run the low-risk reversible ones (push a tag); the truly
                     irreversible/outward ones (cargo publish, store submit, prod
                     deploy) stay manual or need per-action confirmation.             [ask-user]
Phase 5  POST-SHIP   After the operator ships: smoke the released artifact / deployed URL /
                     installed binary; record the version + rollback path; note any
                     platform still in review (decoupled: one platform's limbo never
                     blocks another).                                                  [auto verify]
```

## Button discipline (the one rule that matters)

Reversible & in-repo (version, changelog, build, checksums, preview deploy, drafted
release) → **auto**. Outward & irreversible (publish, prod deploy, store submit,
cargo/npm publish, `git push` of tags/releases, force-push) → **never auto**; surface
as a button with its exact command and rollback. When unsure which side a step is on,
treat it as a button. Ship small + platform-decoupled (the standing cadence): internal →
beta → prod, promote fast when the user base is tiny, and never let one store's review
freeze another platform's release.

## Skills it orchestrates

Map + staged prompts in [references/ROUTING.md](references/ROUTING.md). These release
skills are not part of this pack: use each if installed; otherwise run the same step
directly with git, gh, and your build tool.
Spine: **release-preparations** (tests, version, cross-platform build, checksums, GH
release, installer verify). Plus changelog-md-workmanship · git-commit-craftsman ·
gh-actions · dsr / rch (optional remote-build helpers: GH-Actions fallback when
installed; otherwise local builds or your own CI) · installer-workmanship ·
rust-crates-publishing · gh-cli · vercel / wrangler / gcloud (deploy targets) ·
gh-og-share-images. Pre-ship gates: /bug-zero · security-sweep ·
reality-check · /test-max (test-trust) · /migrate-proof (when the release
carries a migration).

## Shared mechanisms (inherited from project-autopilot)

Green ratchet, decision inbox, capability-aware execution, as in project-autopilot
(see its CONVERGENCE / SAFETY-RAILS / EXECUTION). **Integration:** project-autopilot's
"Release/tag/publish" route points here; ship-it is the lifecycle vertical that closes
build → ... → SHIP, alongside the three quality verticals.

## THE EXACT PROMPT

```
/ship-it
Target: <project path>
Channel: <internal | beta | production | crates.io | app-store | npm | …>
Mode: solo | swarm
```

## Anti-Patterns

- Pressing an outward button (publish / prod deploy / submit / cargo publish / push)
  without the operator's explicit per-action go: the cardinal sin.
- Shipping a red build, or one a reality-check says doesn't deliver.
- Shipping with a secret in the artifact/config.
- Deploying straight to prod instead of a preview first.
- Big-bang hoarding instead of small numbered increments.
- Letting one platform's review limbo block another's release (keep them decoupled).
- Skipping the post-ship smoke (a published thing you didn't verify isn't shipped).

## References

- [ROUTING.md](references/ROUTING.md): phase → skill map, the button list, staged prompts
