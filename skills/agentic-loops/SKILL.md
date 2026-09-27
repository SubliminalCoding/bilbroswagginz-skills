---
name: agentic-loops
description: >-
  A library of proven agentic loop patterns (build/verify/fix/repeat-until-a-bar
  workflows that run a task on a cadence or to a stopping condition), plus the
  anatomy for authoring your own, and a set of ready-to-run loop templates in
  references/loops/. Use when you want to run something iteratively or on a
  schedule, "keep going until X", "run this in a loop", "do this every night /
  every 5 min", a self-improving eval, a maintenance sweep
  (docs/tests/logs/changelog/cleanup), a ticket-to-PR pipeline, an adversarial
  review loop, a champion/challenger optimization, or you ask "what loop fits
  this". Five modes: DISCOVER loop-worthy recurring work in code/history, FIND a
  published loop, AUDIT/repair an existing loop ("loop doctor", "is this loop
  good"), ADAPT one to your tools, or DESIGN/AUTHOR a new one from the shared
  anatomy. Pull the right template, adapt the placeholders, wire it via /loop
  (interval/self-paced) or /schedule (cron cloud). NOT for one-off tasks with no
  repetition or stopping bar.
---

# agentic-loops

> **Core insight:** A good agentic loop is not "do X forever." It is a task wrapped
> in a **verification gate** and a **stopping condition**, that reports **evidence**
> each round. The magic is never the prompt. It's that every iteration *proves* its
> own work before the next one starts, and the loop *knows when to stop*. Replay these
> templates, but more importantly, internalize the anatomy so you can author new ones.

The catalog section below is adapted from the Forward Future Loop Library
(signals.forwardfuture.ai/loop-library), a collection of field-tested loop *prompts*. This skill
makes them pullable, teaches the shape so they generalize, and wires them into THIS
harness (`/loop`, `/schedule`, `Workflow`, Agent builder/reviewer). It also ships
eight ready-to-run, fully-specified templates in `references/loops/`, each with a
goal, cadence, verification gate, and stopping condition already filled in.

---

## The loop anatomy (use this to read OR write any loop)

Every loop below (and every good one you invent) has these five parts. When a loop
"runs away," "never finishes," or "claims done when it isn't," one of these is missing:

1. **Trigger / cadence**: what kicks a round: a schedule (nightly, every 5 min), an
   event (new ticket, production error, customer request), or "just keep going."
2. **Bounded action**: ONE smallest meaningful change per round. Not a batch. The
   smallness is what keeps each round verifiable and reversible.
3. **Verification gate**: proof the round's change is real *before* it counts:
   live-test, run the test suite, render-and-score, revert-or-mutate the fix, ask an
   adversarial reviewer. No gate → slop accumulates.
4. **Stopping condition**: the loop's exit. One of: **streak** (N green in a row),
   **threshold** (metric ≤/≥ target), **budget** (time/iterations/tokens), **dry**
   (a full sweep finds nothing new), **approval** (gated on a human). State it explicitly.
5. **Evidence report**: what each round (and the final stop) hands back: changed files,
   before/after proof, remaining gaps, blockers. "Never describe an errored or exhausted
   run as approved."

**Anti-patterns to refuse:** no stopping condition; folding unrelated refactors into a
bounded fix; claiming done without re-running the original reproduction; silently
truncating coverage to hit a number.

---

## How to actually RUN a loop in this harness

Pick the delivery mechanism by cadence; don't hand-roll what `/loop` or `/schedule` do:

- **Self-paced / "keep going until X"** → `/loop <prompt>` with no interval (model paces
  itself; use for streak/threshold/dry loops where you control the next-step timing).
- **Fixed interval / polling** → `/loop 5m <prompt>` (e.g. the five-minute maintainer,
  production error sweep while watching a deploy).
- **Recurring cloud cron** (nightly changelog, nightly podcast, nightly error sweep that
  must run unattended) → `/schedule` (creates a cron routine). Quote the cadence from the
  loop's trigger.
- **Bounded N-pass refinement** → an apply-the-skill-N-times wrapper, or a `Workflow`
  loop-until-dry / loop-until-budget harness when you want parallel finders + adversarial
  verify per round (only when the user has opted into orchestration).
- **One agent builds, another verifies** (builder/reviewer, devil's-advocate,
  cross-model review) → spawn the reviewer as a separate `Agent` (or worktree) so its
  judgment is independent. For parallel finder fan-out, see `swarm-run`.

Always set the **stopping condition** and the **evidence report** explicitly when you
instantiate; these templates leave them as `[placeholders]` on purpose.

---

## Five ways to use this skill (modes)

Beyond replaying a template, this skill works in five modes. Pick by what the user is
actually asking: not every loop request is "run one."

1. **Discover: "what work here should be a loop?"** Mine the codebase AND the user's
   shell / agent-session history for *recurring* work, then turn the strongest qualified
   candidate into a bounded loop. **Loop-worthiness gate (qualify only if ALL hold):**
   - it has happened **≥2 distinct times** (a one-off is not a loop),
   - **fresh feedback each round can change the next action** (else it's a script, not a loop),
   - **success is verifiable** (a real gate exists), and
   - it has **clear limits + an approval boundary** (a safe, statable stopping condition).
   Route the history-mining to your shell history and `session-mine` (agent sessions);
   this skill supplies the loop-worthiness test + the loop wrap.

2. **Find: "is there a loop for X?"** Match the ask to a template via the routing table
   below (or the ready-to-run set in `references/loops/`). Recommend up to **3** and
   **do not auto-run**: surface them and let the user choose the one + the cadence.

3. **Loop Doctor (audit): "is this loop any good?"** Given a pasted or named loop, find
   the **material** weaknesses and repair only those (no cosmetic rewrites). Audit against
   the anatomy. Three failure modes catch almost every bad loop:
   - **weak/absent verification gate**: it can't prove a round's work before it counts;
   - **unsafe or unbounded action**: a round can do irreversible harm, or batches too
     much to verify/revert;
   - **unclear stopping**: no streak / threshold / budget / dry / approval exit, so it
     runs away or claims done when it isn't.

4. **Adapt: "make this loop fit us."** Tailor a template to the user's real tools,
   permissions, cadence, and definition of success: fill every `[placeholder]`, swap the
   verification gate for what actually runs in this repo, and set the real stopping condition.

5. **Design: "build me a new one."** When nothing fits: ask a few plain-language questions
   (trigger? one bounded round? how is it verified? when does it stop?), then author a short
   bounded loop from the anatomy and wire it via `/loop` or `/schedule`. Don't over-build:
   build the smallest loop that satisfies the five parts.

---

## Ready-to-run templates (`references/loops/`)

Eight fully-specified loops ship with this skill: goal, cadence, `routes_to`,
stopping condition, and verification gate already set. Start here before adapting a
raw catalog prompt:

| Template | One-liner |
|---|---|
| `bug-zero-sweep` | Drive real bugs to zero: verified findings, regression test per fix, stop on two dry sweeps. Routes to `bug-zero`. |
| `dependency-freshness-sweep` | Monthly dep freshness without churn, behind a hard lockfile+test gate. |
| `nightly-changelog` | Keep the changelog tracking what actually shipped, in user language. |
| `overnight-docs-sweep` | Docs always match HEAD: nightly drift catch, de-slopped rewrites. |
| `provider-flip-integrity-loop` | Prove a model flip actually took: live probe, not config faith. |
| `render-farm-throughput-ratchet` | Shard a headless-browser render pipeline to its real fps ceiling. |
| `skillify-harvest-loop` | Weekly: convert recurring friction into registered skills via `skillify` + `session-mine`. |
| `content-from-shipped-work` | Turn shipped work into on-voice posts: traceable artifacts only, human-gated publish. |

---

## The catalog

Placeholders in `[brackets]` are filled per use. Reach for a loop by its **when**, then
adapt the **prompt**.

### Engineering

- **The docs sweep**: when docs have drifted from the code. *Stop:* dry (docs match).
  "Whenever a documentation pass is needed, review the codebase in full and make sure all
  documentation reflects the current implementation. Update stale documentation, verify
  the changes, then open a pull request."

- **The architecture satisfaction loop**: refactor a tangled module to a shape you trust.
  *Stop:* subjective ("happy") + stalled. Tracks state in a tmp file so it's resumable.
  "Refactor until you are happy with the architecture. After each significant step,
  live-test the system, run autoreview, and commit. Track progress in
  /tmp/refactor-{projectname}.md."

- **The sub-50 ms page-load loop**: perf target on every page. *Stop:* threshold (<50ms all pages).
  "Continue optimizing the code for speed. After each significant change, measure
  page-load performance across every page under the same repeatable test conditions.
  Continue until every page loads in under 50 ms."

- **The production error sweep**: triage real prod logs. *Stop:* dry (no actionable errors).
  "Review our production logs for errors. If you find an actionable issue, trace it to its
  root cause, fix it, verify the fix, and open a pull request. If no actionable errors are
  present, stop without making changes."

- **The 100% test coverage loop**: *Stop:* threshold (100%). Pair with a meaningfulness
  guard so it adds real tests, not coverage padding.
  "Add tests until we have 100% test coverage."

- **The logging coverage loop**: instrument every important path. *Stop:* dry.
  "Review the system's logging and add missing coverage until every important path produces
  useful, tested logs."

- **The nightly changelog loop**: cron, user-facing diff. *Trigger:* nightly. *Stop:* nothing shipped.
  "Each night, review changes from the previous day and update the changelog with anything
  users should know."

- **The test-suite speed loop**: faster suite, same coverage/behavior. *Stop:* budget/stalled.
  "Optimize the test suite to run as quickly as possible without reducing coverage or
  changing behavior."

- **The repository cleanup loop**: recover work, kill stale branches/worktrees/PRs. *Stop:* dry.
  "Inspect local and remote branches, pull requests, commits, and worktrees. Recover
  valuable work and clean everything stale until the repository is current and organized."

- **The ticket-to-PR-ready loop**: turn a ticket/bug/complaint into a reviewer-ready patch.
  *Stop:* repro passes + regression green, or "can't reproduce after two attempts." The
  gold-standard bounded-fix discipline: no unrelated refactors.
  "Take a ticket, bug report, failing behavior, or customer complaint and turn it into a
  review-ready patch. Reproduce the failure in the smallest representative environment,
  prove the root cause, make the smallest credible fix, and rerun the original reproduction
  plus relevant regression tests. If the issue cannot be reproduced after two serious
  attempts, say so. Do not fold unrelated refactors into the patch. Finish with the cause,
  changed files, before-and-after proof, risks, and pull-request summary."

- **The cross-model adversarial-review loop**: one model builds, a second independent
  model red-teams the PR. *Stop:* the reviewer approves / only accepted findings remain /
  stalled / iter cap. Needs a second, independent reviewer runtime.
  "Plan the task, implement it, open a pull request, and ask an independent second model
  for an adversarial review. Fix findings above the accepted severity and repeat. Keep the
  branch, PR, findings, verdict, and iteration state resumable. Stop when the reviewer
  approves, only accepted findings remain, progress stalls, or the iteration cap is
  reached. Never describe an errored or exhausted run as approved. Finish with the PR,
  checks, verdict, and remaining findings."

- **The verified-handoff loop**: scheduled repo work (CI triage, dep updates,
  docs sync) where a second session verifies the first. *Stop:* verifier pass, or retry cap.
  "For scheduled repository work such as CI triage, issue grooming, dependency updates,
  or docs sync: set [retry limit], then start an isolated git worktree.
  Let one session stage a patch or outbox message and a second session verify
  it against explicit criteria. Ship only after a pass; otherwise preserve the findings and
  retry only within the limit. Finish with the source revision, staged output, verifier
  result, delivery status, and next run."

- **The fresh-clone loop**: test onboarding from a disposable env, README only. *Stop:* one
  clean clone reaches ready-state / stalled / budget. Carry nothing between attempts.
  "Clone [repository] into a disposable environment and follow only its README to the
  documented ready state, such as running the app or building the package. When a step fails
  or assumes missing knowledge, record the gap, fix the setup or documentation issue,
  discard the environment, and start again. Carry no dependencies, configuration,
  credentials, or repairs between attempts. Stop when one uninterrupted fresh clone reaches
  that state, progress stalls, or [budget] ends. Return exact commands, gaps closed, and
  remaining blockers."

- **The builder-reviewer loop**: builder + reviewer in separate worktrees,
  red-before/green-after test, reviewer proves the test by reverting/mutating. *Stop:* both pass.
  "For [repository task], after the test, build, and lint gates pass: start builder and
  reviewer agents in separate worktrees. The builder reads the shared loop-state file,
  makes one bounded change, and adds a red-before, green-after
  test. The reviewer reruns the gates and proves the test by reverting or mutating the fix.
  Accept only on both passes; park protected or repeated-failure work for a human. Finish
  with the commit, gate evidence, test proof, trust tier, and risks."

- **The completion-contract loop**: define every required outcome + its evidence
  UP FRONT, so partial work can't masquerade as done. *Stop:* all requirements proved.
  "For long-running agent work where partial work could be
  mistaken for done (landing a PR and verifying production is one example), before acting,
  define every required outcome and its evidence. After each bounded action, mark
  requirements proved, weak, missing, or contradicted. Complete the goal only when all are
  proved; otherwise stop as blocked, stalled, or exhausted. Ask before creating goal state.
  Finish with the requirement-to-evidence table, status, owner, and next action."

- **The five-minute repository maintainer loop**: wake every 5 min, one thread per repo,
  highest-value bounded task within granted permissions. *Trigger:* 5-min interval.
  "While repository maintenance is active, wake every five minutes. Triage [repositories]
  and read each repository thread's latest state. Reuse one thread per repository; assign
  its highest-value bounded task only within granted permissions, and do not interrupt
  coherent active work. Require tests, live proof, autoreview, and green CI before work can
  land. Escalate product, access, security, or irreversible decisions. Record meaningful
  changes and stop when every item is landed, decision-ready, blocked, or has no work."

- **The recent-feedback sweep**: mine threads where the user said "this is wrong," dedupe
  into failure patterns, audit the WHOLE project for each. *Stop:* dry / budget.
  "Review all available threads from [lookback window] where I reported something wrong with
  [project] and asked for a fix. Build a deduplicated issue list, group it into failure
  patterns, and verify current state. Audit the complete project for every pattern, fix each
  confirmed instance, and add regression coverage where practical. Repeat the full audit
  until it finds no remaining instance or [iteration budget] ends. Stop on blocked or
  approval-gated work. Return the issues, fixes, evidence, and blockers."

### Evaluation

- **The quality streak loop**: fix-and-restart-the-streak. *Stop:* N successes in a row.
  "Test realistic scenarios. When one fails, document it, add regression and benchmark
  coverage, fix it, and restart the streak. Stop after [N] successful cases in a row."

- **The full product evaluation loop**: N scenarios across every capability, fixed rubric,
  same conditions. *Stop:* every scenario meets the original bar.
  "Create [N] realistic scenarios covering every major capability. Before testing, define
  clear success criteria and choose a consistent evaluation method, such as pass/fail checks
  or a scoring rubric. Run every scenario under the same conditions and record evidence for
  each outcome. Fix the underlying cause of anything that does not meet the criteria, rerun
  the affected scenarios, and then rerun the complete set. Continue until every scenario
  meets the original quality bar."

- **The self-improving champion loop**: champion/challenger on a HELD-OUT set so you don't
  overfit. *Stop:* target / budget / no progress. Promote only on holdout win by [margin].
  "Improve a prompt, policy, or configuration. A support assistant's system prompt is one
  example. Save the champion, its score, a working set, untouched holdout cases, must-pass
  checks, and [budget]. Each round, change one thing based on a recorded failure. Promote
  the challenger only if it beats the champion on holdouts by [margin] without weakening a
  must-pass check; otherwise keep the champion. Stop at the target, budget limit, or no
  progress. Return the winner, scores, experiment log, and remaining failures."

- **The devil's-advocate loop**: a critic argues the design is wrong, logged to
  `.agent-reviews/redteam.md`. *Stop:* no high-impact objection / 2 rounds no new evidence.
  "Before committing to an architecture, interface, or rollout plan, have a critic argue
  that it is wrong. Record each objection, impact, and status in a repository-local log at
  .agent-reviews/redteam.md. The builder must fix and verify each high-impact weakness or
  document why it is accepted; the critic may reopen unsupported answers. Stop when no
  high-impact objection remains or the same issues repeat for two rounds without new
  evidence. Finish with the decision, resolved and accepted objections, evidence, and any
  stalemate."

- **The versioned-experiment loop**: checkpoint + frozen tests/scoring; keep only
  regression-free wins; new revision if the eval changes. *Stop:* success / no progress / budget.
  "To improve a support prompt, code path, or testable subject: in an experiments dir,
  define the goal and [budget], freeze the tests and scoring, checkpoint the current version,
  and record a baseline. Each round, test one hypothesis; keep only a clear, regression-free
  win. If the evaluation changes, open a new revision and rerun the baseline. Ask before
  changing live files. Stop on success, no progress, a blocker, or exhausted budget. Return
  the best checkpoint, comparisons, rollback, and next action."

### Operations

- **The stale-safe batch release loop**: combine valid pending changes, exclude unfinished.
  "Review pending changes and pull requests, exclude stale or unfinished work, combine the
  valid changes, and release them together."

- **The production data cleanup loop**: remove disallowed data + improve the classifier.
  "Review production records, remove anything that does not meet the allowed definition,
  improve the classification logic, and verify the remaining data."

- **The post-release baseline loop**: *Trigger:* release finishes. Re-baseline benchmarks.
  "After current releases finish, run the standard benchmarks and record the results as the
  new baseline."

- **The customer AI deployment loop**: *Trigger:* customer request/failure/ops-review. One
  priority, dry-run on real data, staged rollout. *Stop:* released + monitored.
  "Run this when a customer requests an AI workflow, reports a failure, or reaches an
  operations review. Choose one priority, such as enriching leads, drafting emails,
  summarizing meetings, or updating a CRM. Define the owner, inputs, approvals, success
  metric, and ROI hypothesis. Dry-run it on realistic customer data, fix the smallest
  verified problem, then release through approved stages and monitor production. Finish with
  the outcome, evidence, customer update, lessons saved, and next review."

### Content

- **The SEO/GEO visibility loop**: audit → fix highest-leverage gap → re-crawl/benchmark.
  *Stop:* no critical issues + every priority query maps to an answer-ready page.
  "Run an SEO/GEO audit across crawlability, indexation, page intent, titles, internal
  links, structured data, source citations, and answer-first content. Rank the gaps by
  expected impact, fix the highest-leverage issue, then rerun the same crawl and target-query
  benchmark across search engines and AI answer engines. Repeat until no critical technical
  issues remain, every priority query maps to a clear answer-ready page, and the benchmark
  shows no high-impact gap left to fix."

- **The product update podcast loop**: *Trigger:* nightly. Approved changes → 3–5 min episode
  via a podcast-generation tool. *Stop:* nothing meaningful shipped. Ask before publishing.
  "Each night, review publicly released product changes and select only those users need to
  know. Verify each against the product, docs, or release notes. Use a podcast-generation
  tool to turn the approved changes into a three-to-five-minute podcast explaining what
  changed, why it matters, and how to try it. Check the script and audio for accuracy,
  clarity, and pronunciation. If nothing meaningful shipped, make no episode. Ask before
  publishing. Finish with the draft episode, sources, and review result."

### Design (render-and-score loops)

These share a shape worth stealing: **rig a repeatable capture, score the same views every
round, have a critic name the weakest feature, fix it without regressing the strong ones.**

- **The Boeing 747 benchmark**: Three.js model, 9 fixed angles, rubric. *Stop:* threshold / stalled / budget.
  "Before building, choose reference images, a scoring rubric, [visual threshold], and
  [budget]. Build the most realistic Boeing 747 you can from Three.js primitives, then
  create a rig that screenshots nine repeatable angles. After each change, render and score
  the same views, have a critic identify the weakest feature, and fix it without regressing
  stronger views. Keep the best version. Stop at the threshold, stalled progress, or budget.
  Finish with the model, nine renders, scores, remaining gaps, and run summary."

- **The frontend reconstruction loop**: rebuild a UI from a URL/image, compare at 3
  breakpoints, repair weakest fidelity signal. *Stop:* every gate passes / stalled / blocked.
  "Point the loop at an authorized URL or image. Capture it with a genuine browser and
  record the layout, styles, content, motion, and responsive behavior. Build a static
  mirror and a moving version. Compare both with the source at desktop, tablet, and
  mobile sizes; repair only the weakest fidelity signals. Stop when every gate passes,
  progress stalls, or capture is blocked. Finish with the builds, spec, renders, scores, and
  remaining gaps."

- **The thumbnail refinement loop**: 10 concepts → score at real display sizes → improve
  top 3's weakest dimension → rescore. *Stop:* quality threshold / budget.
  "For [video], use [approved assets] to make ten thumbnail concepts. Score each at real
  YouTube sizes against [inspiration channel] for clarity, curiosity, emotional pull,
  contrast, and accuracy. Take the top three, improve each one's weakest dimension, and
  rescore them under the same rubric. Keep iterating the strongest concept until it clears
  [quality threshold] or [budget] ends. Reject anything the video cannot deliver. Return the
  winner, two runners-up, previews, final scores, and rationale."

---

## Routing: which loop?

| The user wants… | Reach for |
|---|---|
| Docs to match code | docs sweep / `overnight-docs-sweep` template |
| A perf number hit everywhere | sub-50ms (threshold pattern) |
| Prod errors cleaned up | production error sweep |
| Coverage / logging gaps closed | 100% test / logging coverage |
| A nightly unattended job | `nightly-changelog` / podcast / error sweep (via `/schedule`) |
| One ticket → clean PR | ticket-to-PR-ready |
| Independent review of every change | cross-model adversarial review / builder-reviewer / devil's-advocate |
| Long task that mustn't fake "done" | completion-contract |
| Optimize a prompt/policy without overfitting | self-improving champion / versioned-experiment |
| Eval every capability to a bar | full product evaluation / quality streak |
| Make a visual match a reference | Boeing 747 / frontend reconstruction / thumbnail (render-and-score) |
| Find every instance of a reported problem | recent-feedback sweep |
| Test onboarding | fresh-clone |
| Bugs driven to zero | `bug-zero-sweep` template |
| Recurring friction → durable skills | `skillify-harvest-loop` template |

If nothing fits, **author one from the anatomy**: name the trigger, the bounded action, the
verification gate, the stopping condition, and the evidence report, then deliver it via
`/loop` or `/schedule`.

Many loops here have a stronger local equivalent in the installed skill set (e.g.
`bug-zero` and `bug-sweep` for bug sweeps, `ux-maximizer` for render-and-score UI,
`perf-max` for threshold perf loops, `security-sweep` for the security lens,
`skillify` + `session-mine` for the harvest loop). Prefer the richer skill when it
exists; use these templates when you want the lightweight loop itself.
