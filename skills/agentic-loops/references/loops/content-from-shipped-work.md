---
name: content-from-shipped-work
goal: Turn real build activity into publishable, on-voice technical content
cadence: 3x weekly (during a build-in-public push)
run_via: /loop
routes_to: [unslop, beadsmith]
stop_condition: Two weeks of posts are queued, OR there is no meaningful shipped work left to draw from
verification_gate: Each post must name a concrete shipped artifact/lesson/decision (traceable to a commit, release, or bead) and pass a voice + de-slop lint; generic commentary is rejected
---

# Content From Shipped Work

**Goal.** The strongest content source is actual engineering output, not abstract
ideation. This drafts build-in-public posts from what you genuinely shipped, in
your established voice, and refuses to publish hype.

**Each iteration.**
1. Pick one real shipped change or lesson from the window (a commit, a release, a closed bead, a gotcha you just solved; pull from the changelog / git log).
2. Draft a short post in your established voice (load your voice profile or style guide first); run `unslop` to strip hype and AI tells.
3. Drop it into a review/queue state. The outward leg stays human-gated; never auto-publish.

**Stop when** two weeks of posts are queued, OR there's no meaningful shipped work left to draw from this window.

**Verification gate.** Each post must name a concrete shipped artifact, lesson, or decision (traceable to a commit, release, or bead) and pass a voice + de-slop lint. A post that's generic commentary with no concrete artifact is rejected, not queued.

## Copy-paste prompt
```
/loop Turn my shipped work into on-voice posts. Each pass: pick one real shipped
change or lesson from the window (a commit, release, closed bead, or solved gotcha;
source from git log / changelog); draft a short post in my established voice and run
unslop to strip hype and AI tells; drop it into a review queue; never auto-publish.
A post must name a concrete artifact/lesson/decision traceable to a commit, release,
or bead and pass the voice + de-slop lint, or it's rejected. Stop when two weeks of
posts are queued or there's no meaningful shipped work left to draw from.
```

**Cadence / notes.** Runs during a build-in-public push. Pairs with [nightly-changelog](nightly-changelog.md) as the source feed. The outward leg stays human-gated.
