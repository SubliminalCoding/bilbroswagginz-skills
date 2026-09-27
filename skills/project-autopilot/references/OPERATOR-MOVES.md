# OPERATOR-MOVES.md: The Operator's Project-Pushing Playbook

> **Source note:** These moves were mined from real Claude Code session history
> across several projects (a consumer mobile app, a web platform, a multi-tenant
> SaaS, a game prototype). The examples are paraphrased; project names and
> identifying details have been removed.

This file is the DECIDE step's brain. At each tick, find the move whose trigger
matches the current project state and execute it the way the operator would.

## Loop integration: which move fires at which tick state

| Tick state | Move |
|---|---|
| First tick, fresh session | Cold-open deep orient / Re-entry catch-up |
| Anything red in the live build | Fix the broken thing before building the new thing |
| Gap chosen, multiple viable routes | Recommend-then-commit decision loop |
| Gap touches pricing/API costs/recurring spend | Business-margin gut check → **ask-user** with the math attached |
| Mid-run, quality plateau, early-stage | Ambition spike (gated by late-stage rule) |
| Surface works but unhardened | Skill-stacked quality gates, run in multiples |
| Big analysis/audit skill (subagent fan-out) just finished | Smart-model self-review: re-check the recommendations |
| Skill reports "done" | Verify by running it: claims don't count |
| New evidence available (screenshots, logs) | Screenshot punch-list iteration: milk it dry |
| About to mutate something that works | Checkpoint before risk |
| A gap is dragging or threatens stability | Scope cut without sentimentality → inbox if judgment |
| Long-running operation started | Background the waits, stay in motion |
| Verified batch accumulated | Ship in small numbered increments (**ship itself = ask-user**) |
| Live-user breakage detected | Production fire drill (preempts everything) |
| Commercial project, launch phase | User-pulse check |
| Project functional + structurally settled, pre-ship (NOT early build) | Late-stage optimization pass (profile → repeatedly optimize) |
| Dry passes beginning | Convergence clamp (review-only endgame) |

## Voice & posture

- **Decisions arrive in under ten words.** "ok do it", "option b", "yes ship it", "go", "a". The operator never writes a paragraph when a syllable commits the work. Long messages are only for *describing observed reality* (bugs, screenshots, user feedback) or *thinking aloud through a genuine tradeoff*.
- **The operator delegates the HOW completely, holds the WHAT and WHETHER.** The dominant loop: pull a recommendation out of the agent, sanity-check it against business reality (margins, users, risk to the live app), then commit instantly. They almost never specify implementation.
- **The operator is the QA department.** No claim of "done" counts until they have personally installed the build on a real device and poked at it. They report findings as screenshot batches with run-on punch lists.
- **The working product is sacred.** Novelty never justifies risk to something live. They checkpoint before big changes, revert without sentimentality, and name their own failure mode out loud: the risk of overbuilding.
- **Skills are the operator's power tools, run in multiples.** They invoke named skills explicitly, with a scope and a count (a set number of UI polish passes, a deep bug hunt), and ask which other skills could help when momentum stalls.

---

## Cold-open deep orient

- **Trigger:** Starting serious work on a project the agent hasn't internalized this session (new session, new repo, or pivoting into a new work phase).
- **The move:** Force total context ingestion before any action: read the project docs completely, map the architecture, then immediately chain into a named skill pipeline so orientation flows straight into work.
- **In practice:** ask the agent to read AGENTS.md and README.md in full, use its code-investigation mode to map the architecture and purpose, then run `hotspot-scout` and start optimizing with `optimize-proof`. The same opener was reused across several projects.
- **Anti-move:** Letting the agent start editing from a shallow skim, or orienting without a concrete next skill queued behind it.

## Re-entry catch-up

- **Trigger:** Returning after hours/days away, or resuming a compacted session.
- **The move:** Demand a state briefing before anything moves: where were we, what's in flight, what's the next concrete step.
- **In practice:** after time away, ask the agent to summarize where a multi-step process (such as an app-store submission) stands and what the next step is, or simply ask it to pick up where the last session left off.
- **Anti-move:** Re-planning from scratch or starting new work before reconciling with the last session's in-flight state.

## Fix the broken thing before building the new thing

- **Trigger:** A planned feature push collides with a known defect in the current build.
- **The move:** Park the roadmap; diagnose and fix the live breakage first. New features land on a working base only.
- **In practice:** agree to the new features, but first diagnose why a core feature is currently broken; when one platform is blocked, pause work there and confirm the other platform's build actually works.
- **Anti-move:** Stacking features on a known-broken foundation because the roadmap says so.

## Recommend-then-commit decision loop

- **Trigger:** Any fork in the road: options exist, and the agent knows more about the tradeoffs than the operator does.
- **The move:** Pull a single clear recommendation (not a menu), pressure-test it against industry norms and business math, then commit in one line and don't relitigate. (In autopilot: auto-routes commit themselves this way; ask-user routes present ONE recommended option first in the inbox entry.)
- **In practice:** ask for the agent's recommendation (the most frequent single request in the history), push back if the answer is hedged until it is stated plainly, check whether it matches industry standard, then lock it in with a one-line "go with your recommendation."
- **Anti-move:** Presenting 5 unranked options, hedging, or re-opening a locked decision without new evidence.

## Business-margin gut check

- **Trigger:** Any decision that touches pricing, API costs, or recurring spend.
- **The move:** Before committing, run the unit economics out loud: what does this cost per user, what's the margin at current pricing, can the free tier eat it. In autopilot this is always **ask-user**, but the inbox entry must arrive with the math already done.
- **In practice:** ask whether the product stays profitable at the current price after provider and store fees, flag a well-liked but expensive feature (such as premium voice) whose costs could pile up, and raise the price or cap the feature when unlimited use can't be covered.
- **Anti-move:** Shipping a cost-bearing feature (voice, premium LLM) without a cap or a margin story.

## Ambition spike (the "Wow factor" push)

- **Trigger:** Product is functional but feels merely adequate; energy for a step-change in quality. GATED: never mid-iteration on a late-stage build (>10 modified files OR >3 in-flight beads) without asking.
- **The move:** Run an idea-forge maximal creativity pass framed at premium/world-class level, then immediately funnel the winners into a tracked backlog and execute serially.
- **In practice:** ask for an idea-forge pass on turning the product into a premium experience with a "wow" factor, covering every facet of the app and applying real taste; then approve the winners, create beads for each, and work through them one by one.
- **Anti-move:** Letting a brainstorm end as a list. Ideas that survive ranking go straight into beads and get executed one by one, never mass-dispatched.

## Skill-stacked quality gates, run in multiples

- **Trigger:** A surface works but hasn't been hardened; pre-release; or after a heavy build sprint.
- **The move:** Fire named quality skills at a specific scope, in explicit counts, often stacked back-to-back: polish passes, audits, bug hunts, de-slop. (deepen is the executor for counted passes.)
- **In practice:** request a set number of UI polish passes on specific pages or on one screen, apply the agent's recommendations and then run two named bug-hunting skills twice each, and make sure any AI-generated user-facing text gets a de-slop pass.
- **Anti-move:** One generic "review the code" pass. The gates are named, scoped to a screen/feature, and repeated N times.

## Honest-take extraction

- **Trigger:** Risk of agreeable drift; a taste/quality judgment is needed rather than execution.
- **The move:** Explicitly license brutal candor and ask for critique of a specific artifact.
- **In practice:** ask for an honest audit of a specific prompt, for the agent's real opinion of the app, or for its candid take on a concrete design change such as removing a header from every page.
- **Anti-move:** Flattery, or critique without a concrete fix attached.

## Verify by running it: claims don't count

- **Trigger:** A skill or agent reports a fix or feature complete.
- **The move:** Test before commit, test after deploy, and follow the full user journey to its end. Where possible test synthetically (the loop's smoke check); device-level verification queues for the operator.
- **In practice:** require a local test before committing, ask the agent to test the flow itself with synthetic sessions, insist the journey is followed to the end, check the logs first, and ask to see the actual edited file rather than a description of it.
- **Anti-move:** Trusting "the tests pass" as proof the experience works. In one project, dozens of messages were variants of "still not working": re-test after every claimed fix until it actually works.

## Screenshot punch-list iteration (and milk the evidence dry)

- **Trigger:** Fresh evidence is available: screenshots, logs, user transcripts.
- **The move:** Extract a defect list, fix the named items, then re-interrogate the *same* evidence for more before asking for new evidence.
- **In practice:** have the agent analyze a batch of screenshots, fix what it found, then ask what else it sees in the same images, fix those too, and ask again.
- **Anti-move:** Treating one pass over evidence as exhaustive. The same screenshot gets squeezed three or four times.

## Smart-model self-review (don't trust the subagents)

- **Trigger:** A big analysis/audit skill that fanned out to subagents just finished (security audit, codebase audit, bug hunt, optimization sweep). Its recommendations are sitting there waiting to become beads.
- **The move:** Before ANYTHING becomes a bead, the top-level (smartest) model independently re-reviews every top recommendation against the actual code and forms its own judgment: agree or not, to what degree, what to actually do. The analysis subagents are often smaller models without full project context; the smart model re-checking catches false positives, over-reach, and context-blind advice before it costs a bead or a commit.
- **In practice:** tell the top-level model to review each top recommendation itself, dig into the actual code, and say whether it truly agrees, which ones, why or why not, to what extent, and what it thinks should actually be done.
- **Why it's critical:** this is the single highest-leverage gate in the big-skill pipeline (EXECUTION.md). The operator calls it out as non-skippable specifically *because* the cheap models do the fan-out work. It's the heavyweight sibling of LENSES' "re-verify before work."
- **Anti-move:** Piping subagent findings straight into beads or code because they're already written up. Volume of findings ≠ correctness of findings.

## Checkpoint before risk

- **Trigger:** About to make a large or identity-level change to something that currently works (prompt overhauls, redesigns, big refactors).
- **The move:** Save the current state somewhere recoverable *first* (git, memory files, copies of prompts) and state the rollback path out loud. (This is the green ratchet's ancestor; in autopilot it's mandatory per CONVERGENCE.md.)
- **In practice:** before changing the product's style or prompts, save the originals so they can be restored; before another build, save what works so far, naming the risk of overbuilding.
- **Anti-move:** Letting a redesign overwrite the only copy of what works. (On one live web app the big-change experiment was tried *and rolled back the same day*; the checkpoint made that cheap.)

## Scope cut without sentimentality

- **Trigger:** A feature is dragging, confusing users, or threatening the stability of the live product; or an exciting tangent appears mid-launch.
- **The move:** Kill it, defer it, or revert it in one line. Working-and-shipped beats ambitious-and-wobbly, every time. In autopilot: stability-threatening gaps get cut/reverted (auto); judgment cuts (removing a feature users might want) go to the inbox with a recommendation.
- **In practice:** when too much was added, revert it; roll back the site for now; skip a pricing tier for now; and once something works, keep it working for a while before any large change.
- **Anti-move:** Sunk-cost defense of built features, or letting a strategic tangent leak into the shipping product's codebase. Those get split into a separate plan doc instead.

## Background the waits, stay in motion

- **Trigger:** Any long-running operation: builds, deploys, CI, store review.
- **The move:** Push the wait into a background monitor with a callback, then immediately switch to parallel non-mutating work (docs, copy, the next diagnosis). Never sit idle watching a progress bar.
- **In practice:** ask the agent to monitor the build and report when it lands, and while it builds, move on to fixing the store listing copy.
- **Anti-move:** Serializing the project around build times. (One consumer-app session history shows 100+ background monitor events interleaved with store setup, pricing work, and bug fixes happening simultaneously.)

## Ship in small numbered increments

- **Trigger:** A coherent batch of fixes/features has accumulated and verified.
- **The move:** Cut a numbered build, push through the smallest safe channel first (internal → beta → production), promote fast when the user base is tiny, keep platform tracks independent. In autopilot: preparing the release = auto-prep via release-preparations (if installed; otherwise prep it by hand: tests, version bump, build); pressing the button = **ask-user**, always.
- **In practice:** cut a numbered build, submit it to the internal track, and once it checks out, promote it to production; with few users, skip the staged percentage rollout and release to everyone at once.
- **Anti-move:** Hoarding changes for a big-bang release, or letting one platform's review limbo freeze the other's shipping.

## Production fire drill

- **Trigger:** A real user reports breakage, or live monitoring shows failures. PREEMPTS EVERYTHING.
- **The move:** Drop everything, root-cause it ("get to the bottom of this"), fix all the way (not just the symptom), then institutionalize the fix so it can't recur, and save the playbook to memory (cm learning).
- **In practice:** when new users can't log in after install, have the agent investigate and find the root cause, then ask how to make sure it never happens again, then have it save the fix to memory for future reference.
- **Anti-move:** Patching the symptom and moving on. The sequence is always fix → prevention → memory.

## User-pulse check

- **Trigger:** Daily-ish during launch phase of a commercial project; after any marketing action; when deciding what to build next.
- **The move:** Pull real usage numbers and read actual user sessions. Let live behavior, not the roadmap, drive the next move. (Distribution-lens findings route here; mostly produces inbox items.)
- **In practice:** check for new users nearly every day, look into who a new paying customer is and how they arrived, and ask how many downloads turn into actual active use.
- **Anti-move:** Vanity-metric satisfaction. Installs ≠ usage; individual transcripts reveal why users bounce (one product's prompt-style pivot came directly from session reading).

## Late-stage optimization pass (profile, then repeatedly optimize)

- **Trigger:** The project is functional and structurally settled: core vision goals are WORKING, the correctness lens is green, and the codebase is no longer churning on new features. This is a convergence / pre-ship maturity activity, NOT an early-build one. Also fires on the operator's explicit ask. **Gate it: do not run this early.**
- **The move:** The operator's two-step, run as a chain. **Step 1 (orient + profile):** re-read AGENTS.md and README fully, deep-investigate the architecture, then run `hotspot-scout` to produce a *scored* hotspot list under real-world usage. **Step 2 (repeatedly optimize):** wrap `optimize-proof` in `deepen` and grind the top-ranked hotspots systematically, one at a time, each optimization behavior-proofed and held to the green ratchet (a "faster" change that breaks a test is reverted, not kept).
- **In practice:** open with the cold-open orient (read AGENTS.md and README.md in full, map the architecture), run `hotspot-scout` to find hotspots under real-world usage, start fixing them with `optimize-proof`, then follow up by repeatedly applying `optimize-proof` through `deepen` to the critical items the profiling found.
- **Why the gate:** profiling a still-changing codebase optimizes code that's about to be rewritten: wasted passes, and worse, it can lock in structure the project hasn't earned yet. Profile a working thing you're about to ship, not a half-built one. (This is the disciplined form of "premature optimization is the root of all evil": the optimization is welcome; doing it early is the mistake.)
- **Anti-move:** Reaching for `optimize-proof` during active feature build, or optimizing without a profiler-scored target list (optimizing by vibe instead of by measured hotspot).

## Convergence clamp (review-only endgame)

- **Trigger:** Work has converged; remaining risk is reviewers/agents inventing new work to justify themselves. This IS the dry-pass mechanism in CONVERGENCE.md.
- **The move:** Flip to blocker-only mode with an explicit evidence bar and a permission to stop.
- **In practice (orchestrating a project's review agents):** ask for one final quick blocker check only, with no full test run, and have agents answer NO BLOCKERS unless they can cite a new concrete blocker with file/line evidence.
- **Anti-move:** Open-ended "any other thoughts?" at the end of a project. It manufactures infinite non-blocking findings.

---

## Cadence model: an operator-driven project from kickoff to shipped

1. **Orient hard:** cold-open mega-prompt (read ALL the docs, understand the architecture) chained directly into a profiling/audit skill, or a re-entry catch-up.
2. **Stabilize first:** anything broken in the live build gets diagnosed and fixed before new work starts.
3. **Build in recommend→commit loops:** one clear recommendation → one-line commit → keep-going throttle while output is good. Long waits go to background monitors; parallel non-mutating work continues.
4. **Spike ambition periodically:** "Wow factor" idea-forge pass → rank → beads → knock them out ONE by ONE. (Never mid-iteration on a late-stage build without asking.)
5. **Gate quality in multiples:** named skills, counted passes: several UI polish passes, two bug-hunting skills run twice each, a de-slop pass on anything user-facing.
6. **Verify for real:** run the numbered build, walk the full user journey to the end, milk each evidence batch (ask what else the agent sees), repeat until it actually works.
7. **Optimize once it's settled:** when the thing works and the structure has stopped moving: orient + `hotspot-scout` for a scored hotspot list, then `deepen` over `optimize-proof` on the top hotspots. Late-stage only, never on a still-churning build.
8. **Checkpoint, then ship small:** save state before anything risky; cut numbered builds; internal → production fast; platforms decoupled. (Ship button = the operator's.)
9. **Watch the pulse:** daily user counts, real session transcripts, paying-customer forensics. Production fires preempt everything: root cause → prevent → save to memory.
10. **Converge with a clamp:** blocker-only review with file/line evidence bar, explicit permission to answer NO BLOCKERS, then move on to selling the thing.
