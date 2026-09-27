# Skill Routing Map

Maps each gap type the lenses can surface to the installed skill that executes the
fix, and classifies every route as **auto** (the loop runs it unattended) or
**ask-user** (queued to the decision inbox; the loop works around it).

Skills marked † are not part of this pack: route to them if installed; otherwise
do the fix directly under the same batch and ratchet rules (see DEPENDENCIES.md at
the pack root).

## Classification rule (apply when a gap doesn't match a row)

**auto** = pure engineering inside the repo, reversible, verifiable by tests/build.
**ask-user** = any of: outward-facing (publishes, releases, sends, charges),
irreversible, pricing/brand/scope judgment, touches money/secrets/credentials,
or contradicts a documented operator preference. When in doubt → ask-user. A wrongly
queued auto item costs one review; a wrongly auto'd ask-user item costs trust.

## Routing algorithm

1. Most specific skill wins (e.g. `security-sweep` over generic `audit-lens` for billing code).
2. Honor "When to Use What" tables inside the skills themselves; they cross-reference each other.
3. One skill per tick. Never stack two mutating skills in a single batch.
4. No match → `research-software`† to investigate (auto; if absent, investigate directly), and log a "new skill candidate" to the run report (creation = ask-user).

## Correctness

| Gap signal | Skill | Mode |
|---|---|---|
| **Exhaustive correctness pass / "fix all the bugs" / pre-ship hardening** | **/bug-zero** (the correctness vertical: orchestrates the finders below + adversarial verify + regression-test-per-fix + loop-until-dry) | auto |
| Symptom can't be reproduced on demand ("sometimes", "only in prod", heisenbug) | trace-it: instrument first, capture the bad state, hand the located cause to the fixers | auto |
| LIVE production incident: users seeing it NOW | incident-zero (stabilize → evidence → fail-first regression → prevention; post-incident cleanup lands back at /bug-zero) | reversible stabilization auto; **irreversible levers ask-user** |
| Bugs suspected / pre-release hardening | bug-sweep | auto |
| Pre-commit scan, AI-code validation | ubs-workflow† (ubs) | auto |
| Stubs, mocks, placeholder/fake code in prod paths | mock-code-finder† | auto |
| Hangs, flaky tests, races, deadlocks | deadlock-finder-and-fixer† | auto |
| Process crash/spin needing live debugging | gdb-for-debugging† | auto |

## Performance

| Gap signal | Skill | Mode |
|---|---|---|
| **Late-stage optimization pass** (project functional + settled, pre-ship) | **/perf-max** (the performance vertical: profile → optimize → behavior-prove → measure → re-profile, the chain below wrapped in proof + the late-stage gate) | auto, **late-stage only** (see gate) |
| "Slow" but unprofiled: rank hot paths first | hotspot-scout | auto |
| Scored hotspot list ready to optimize | optimize-proof | auto |
| Repeated optimization over the ranked hotspots | deepen (wrapping optimize-proof) | auto, late-stage |
| Host machine itself bogged down | system-performance-remediation† | auto |

**Late-stage gate (the two-step optimization pass):** the profile→repeatedly-optimize
chain runs ONLY when the project is functional and structurally settled: core goals
WORKING, correctness green, features no longer churning. Profiling a still-changing
codebase optimizes code that's about to be rewritten. Treat it as a convergence-phase
activity, not an early-build one; it's the payoff stage of the "Cold-open deep orient"
move. Full move + example prompts: OPERATOR-MOVES "Late-stage optimization pass."
Each optimization stays under the green ratchet: a faster change that reds a test is
reverted, not kept.

## UX / UI

| Gap signal | Skill | Mode |
|---|---|---|
| **Exhaustive UI/UX maximization / "make it premium" / wow-factor** | **/ux-maximizer** (the UI/UX vertical: idea-forge wow → top 5 → 3× ux-scan → 5× polish-pass → 3× design-intel → loop) | auto (taste calls ask-user) |
| Usability/heuristic evaluation | ux-scan | auto |
| Works but needs visual elevation | polish-pass | auto |
| New UI surface to design/build | design-intel | auto |
| Desktop-first app broken on phones | mobile-retrofit-for-webapps† | auto |
| Diagrams, charts, sims needed | interactive-visualization-creator† | auto |
| TUI rendering/regression | tui-inspector†, frankentui† | auto |

## Security

| Gap signal | Skill | Mode |
|---|---|---|
| General security sweep | audit-lens (domain: security) | auto (findings + code fixes) |
| Billing/payment/webhook security | security-sweep | findings auto; payment-flow changes **ask-user** |
| Secrets found in code/chat history | fix per no-secrets rule | rotation = **ask-user** (needs the operator's accounts) |

## Architecture & code quality

| Gap signal | Skill | Mode |
|---|---|---|
| Unfamiliar codebase, need mental model | codebase-archaeology† | auto |
| Bloat, duplication, needless complexity | simplify-and-refactor-code-isomorphically† | auto |
| Complexity IS the finding (god-function, unreadable module): reduce it without changing behavior | simplify-proof (characterize FIRST → one transform at a time → measured complexity drop or revert; wraps the isomorphic-refactor mechanics as its gate) | auto |
| Stale deps | deps-update† | patch/minor auto; **major bumps ask-user** |
| Schema changes | database-migration-expert† | additive auto; **destructive ask-user** |
| Migration where old and new must coexist (schema/data/framework/service cutover) | migrate-proof (expand → migrate → contract; parity oracle + rehearsed rollback gate the contract; drives database-migration-expert† for DB mechanics when installed) | expand auto; **contract ask-user** |
| Proven-but-bloated app needing v2 | clean-room-v2-rebuild† | **ask-user** (scope decision) |

## Copy, docs, messaging

| Gap signal | Skill | Mode |
|---|---|---|
| README missing/stale/overselling | readme-real | auto |
| AI-slop tells in docs/UI copy | unslop | auto |
| Changelog absent or rotted | changelog-md-workmanship† | auto |
| Public docs site | documentation-website-for-software-project† | build auto; **deploy ask-user** |

## Testing

| Gap signal | Skill | Mode |
|---|---|---|
| **Exhaustive test-trust pass / "can we trust this suite?" / weak or flaky tests** | **/test-max** (the testing vertical: behavior inventory → gap-fill with the testing-* toolkit below → mutation-proof gate → flake-zero) | auto, **late-stage** (features settled, like the perf pass) |
| Webapp lacks E2E coverage | e2e-testing-for-webapps† | auto |
| Mocked tests hiding real-service breakage | testing-real-service-e2e-no-mocks† | auto |
| Parsers/protocols/serialization untested | testing-fuzzing† | auto |
| Spec/port conformance | testing-conformance-harnesses† | auto |
| Output-stability regression risk | testing-golden-artifacts† | auto |

The `testing-*`† family is the late-stage **testing-posture** toolkit. Once features
are settled, apply each *to the extent it's applicable* to harden coverage
(real-service-no-mocks, conformance, golden artifacts, fuzzing, metamorphic). Like
the optimization pass, this is a maturity activity, not early-build work. When the
testing lens fires suite-WIDE (green that can't be trusted, not one missing harness),
route to test-max: it conducts this toolkit and adds the mutation-proof trust gate,
exactly as the correctness lens routes to bug-zero.

## Product & strategy (decision-prep runs auto; the DECISION is the operator's)

| Gap signal | Skill | Mode |
|---|---|---|
| Vision vs reality drift | reality-check | auto (this IS the assess step) |
| What to build next / feature ideation | idea-forge (high-stakes / direction-setting → dueling-idea-forge) | early-stage: generate→beads auto, execute via normal batches; late-stage (>10 modified files OR >3 in-flight beads): **ask-user** per standing rule |
| Direction/pricing/scope/kill calls | dueling-idea-forge (adversarial decision brief: competing takes argued, scored, synthesized) | runs auto to produce decision brief → **ask-user**; the operator decides |
| Pricing, monetization, offers | (none) | always **ask-user** (brief prepared by dueling-idea-forge) |

## Process & infra

| Gap signal | Skill | Mode |
|---|---|---|
| Plan → tracked work | beadsmith (br) | auto. Note: `br` has no `comment` subcommand; put closure evidence in the commit message or bead description |
| Triage/prioritize backlog | beads-bv† (or `bv` directly) | auto |
| Parallel execution wanted | swarm-run (sole-writer ownership map + orchestrator-re-run gate: the discipline layer), with ntm† + vibing-with-ntm† as the pane tooling | auto **only if autopilot launched in swarm mode**; solo mode never self-escalates to a swarm. Tending shape (4 cod + 2 cc, looper, rch, stalled-bead recovery) = SAFETY-RAILS "Swarm-mode tending" |
| Review phase saturated but a different CLASS of work is suspected | fresh diagnosis sweep: mock-code-finder† · deadlock-finder-and-fixer† · reality-check · modes-of-reasoning-project-analysis† · hotspot-scout (late-stage) · security-sweep; each → smart-model review → beads | auto (per CONVERGENCE saturation gate) |
| CI missing/broken | gh-actions† | auto |
| Commit hygiene | git-commit-craftsman† | auto |
| Release/tag/publish/deploy-to-prod | **/ship-it** (the release vertical: does all reversible pre-flight, hands the operator the outward buttons) | **ask-user** (outward-facing; reversible pre-flight auto) |
| Deploy (Vercel/Railway/etc.) | vercel†, wrangler†, gcloud† | preview auto; **production ask-user** |

## Meta

| Gap signal | Skill | Mode |
|---|---|---|
| No installed skill fits | research-software† → log new-skill candidate | research auto; creation **ask-user** |
| Repeated multi-pass refinement of one facet | deepen | auto (bounded by CONVERGENCE.md) |
| Need prior-session context | `session-mine` + `memory-forge` | auto |
