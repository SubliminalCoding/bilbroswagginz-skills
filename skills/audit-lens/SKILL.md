---
name: audit-lens
description: >-
  Domain-parameterized codebase audit: assess a codebase through ONE chosen lens
  (API design, CLI ergonomics, data integrity, general code quality, or a broad
  multi-domain sweep) and return ranked, verified, located findings each with a
  concrete fix. For domains that have a dedicated specialist (security →
  security-sweep, performance → hotspot-scout, UX/a11y → ux-scan, bugs →
  bug-zero, copy/slop → unslop) it ROUTES instead of redoing the work.
  Use when "audit this", "assess the code quality", "review the API design",
  "CLI ergonomics pass", "data-model review", "pre-launch audit", "full audit
  across the board", or when project-autopilot / forge / ship-it dispatch
  "audit-lens (domain: X)".
---

# Audit Lens

Point it at a codebase, name a lens, and get back what a senior reviewer in that
discipline would hand you: a ranked list of verified, located findings, each with
file:line, why it matters here, and the concrete fix. It is the pack's *generalist*
auditor, and it knows exactly what that means: for the four domains nobody else in
the pack covers (API design, CLI ergonomics, data integrity, general code quality)
it does deep native work; for everything a specialist covers, it dispatches the
specialist and gets out of the way. Low-risk fixes can land under a green ratchet;
judgment calls go to the decision inbox, never guessed.

> **Core Insight:** The most valuable thing a generic auditor can do is *refuse to
> run*. The pack has specialists that beat any generic pass in their own domain:
> security-sweep proves exploit paths, hotspot-scout measures instead of guessing,
> ux-scan walks the UI like a screen-reader user. A generic "security audit" next
> to security-sweep is strictly worse output wearing the same label, and two skills
> auditing the same domain produce conflicting findings that someone has to
> reconcile. So the routing rule is **most-specific-skill-wins** (the same rule
> project-autopilot already encodes): on dispatch, check the domain first. If a
> specialist owns it, return a routing pointer and let the specialist's contract
> apply. Audit-lens only burns tokens where it is the most specific skill available.

## Preconditions / guards

1. **Route before you read**: resolve the `Domain:` parameter against the routing
   table FIRST. Mapping to a specialist costs one table lookup; a duplicate generic
   audit costs a full pass that loses to the specialist anyway.
2. **Ground before you sweep**: for native domains, read how the surface is
   actually built (route handlers vs. server actions, clap vs. hand-rolled args,
   migrations vs. drifted schema) before grepping. A finding written against the
   wrong mental model is a phantom.
3. **Green ratchet on any applied fix**: a fix that reds a test reverts. Never
   weaken a test to pass.
4. **Judgment calls are ask-user**: anything that changes a public API shape, a
   CLI flag users may script against, a schema/migration, or product behavior →
   write finding + recommended fix to `<project>/.autopilot/DECISIONS.md`
   (recommended option first) and DO NOT auto-edit. Breaking a consumer to "fix"
   a style finding is worse than the finding.
5. **Read-only against real services**: audits inspect; they don't fire requests
   at prod, run migrations, or mutate data to "confirm".

## The routing table (Phase 0: most specific skill wins)

| Domain requested | Who runs it |
|---|---|
| security · billing · auth · webhooks · RLS · secrets · tenant isolation | **security-sweep** |
| performance · hotspots · latency · "why is it slow" | **hotspot-scout** (then **optimize-proof** to grind the list) |
| UX · usability · accessibility · a11y | **ux-scan** |
| bugs · correctness · "find everything broken" | **bug-zero** (one specific bug → **no-slop-bug-hunter**) |
| copy · messaging · docs voice · AI-slop | **unslop** |
| API design | audit-lens, native |
| CLI ergonomics | audit-lens, native |
| data integrity / data-model | audit-lens, native (RLS/isolation aspects still → security-sweep) |
| general code quality | audit-lens, native |
| broad multi-domain sweep | audit-lens orchestrates: specialist domains dispatched, native domains run here, ONE merged ranked report |

The rule is open-ended: if a *more specific* skill for the requested domain is
installed (check the skill list), route to it even if this table predates it. A
routed domain returns a pointer (skill name + the exact invocation), not findings;
the specialist's own contract (its guards, its inbox behavior) applies from there.

## The Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  ROUTE    Resolve Domain: against the table. Specialist exists →
                  emit "ROUTED → <skill>" + its exact invocation, STOP (for a
                  broad sweep: dispatch specialists, continue with native
                  domains only). No specialist → continue.
Phase 1  MAP      Ground in the real surface for the lens (guard #2): every
                  public route/endpoint for API design; every subcommand, flag,
                  and exit path for CLI ergonomics; every table, constraint,
                  migration, and write path for data integrity; the module map
                  for code quality. This map bounds the audit: findings cite
                  it, and coverage is measured against it.
Phase 2  SWEEP    Walk the domain rubric (references/RUBRICS.md) against the
                  map. For each candidate finding, write the FAILURE STORY: who
                  hits this (API consumer, scripter, future migration, next
                  maintainer), doing what, and what breaks or degrades. No
                  failure story = not a finding yet.
Phase 3  VERIFY   Adversarially re-check EVERY candidate against the real code:
                  open the cited file:line, confirm the problem isn't handled
                  one layer up (shared middleware, a clap derive default, a DB
                  constraint, a wrapper the grep missed). Default to "not a
                  finding" if the failure story doesn't survive. Doubtful +
                  high-stakes → a second model (multi-model-triangulation if installed). Phantoms die HERE.
Phase 4  RANK     Order survivors by impact × likelihood: breaks-consumers /
                  corrupts-data first, then silently-wrong-results, then
                  friction-at-every-use, then polish. Severity is the failure
                  story's damage, not the rubric item's name.
Phase 5  REPORT + REMEDIATE  Every survivor into the ranked report. Then split:
                  • Low-risk mechanical fixes (error envelope on an internal
                    route, missing --json flag, stderr/stdout mix-up, a dead
                    module) → fix under the green ratchet, regression test per
                    fix where one can fail on the old code.
                  • Public-API shape, scriptable CLI surface, schema/migration,
                    product behavior → DO NOT EDIT; finding + recommended fix
                    to .autopilot/DECISIONS.md (guard #4).
```

## Output (what orchestrators get back)

- **Routing pointer**: for specialist domains: `ROUTED → <skill>` + the exact
  invocation, so the dispatcher re-fans-out without re-deriving the mapping.
- **Ranked verified findings**: for native domains: each with lens, severity,
  the failure story, cited file:line, and the concrete fix.
- **Applied fixes**: low-risk only, green-ratcheted, regression-tested.
- **`<project>/.autopilot/DECISIONS.md`**: appended judgment items, recommended
  option first. Same inbox project-autopilot, idea-forge, and security-sweep
  drain. One inbox, not four.
- **Merged report**: broad sweep only: specialist results + native results in
  one ranking, each finding tagged with the skill that produced it.

## Contract (what orchestrators rely on)

- **project-autopilot** routes "General security sweep | audit-lens (domain:
  security)" here; per most-specific-skill-wins this returns the security-sweep
  pointer rather than a duplicate generic pass; the autopilot's fan-out lands on
  the specialist either way. Native-domain dispatches return ranked findings AUTO,
  judgment changes ask-user via the inbox.
- **forge / ship-it** use it as the pre-ship audit gate for the native domains and
  as the router for everything else.
- Takes a `Domain:` parameter; `Domain: all` (or none) = broad multi-domain sweep.
- **Most specific skill wins**: never produces findings in a domain a specialist
  owns.

## Skills it composes with

Routes to **security-sweep**, **hotspot-scout** (+ **optimize-proof**),
**ux-scan**, **bug-zero** / **no-slop-bug-hunter**, **unslop**. Verifies
doubtful findings with **multi-model-triangulation** (if installed; otherwise a
second model by hand); **supabase** (if installed) for schema / RLS specifics when
the data-integrity lens touches Postgres; files large finding sets as a **br**
graph (label `audit-lens`, if br is installed; otherwise a markdown list) when the
user wants them tracked instead of read; **reality-check** first if the project's intent is unclear (an audit
without a target intent ranks blind).

## THE EXACT PROMPT

```
/audit-lens
Target: <project path or subsystem (e.g. the public API routes, the CLI crate)>
Domain: <security | performance | ux | bugs | copy | api-design |
         cli-ergonomics | data-integrity | code-quality | all>
Stack:  <e.g. Next.js + Supabase + Stripe | Rust CLI | Electron>  (optional; it will infer)
Fix:    <report-only | fix-low-risk>   (default: fix-low-risk)
```

## Anti-Patterns

- Running a generic security/perf/UX/bug/copy pass "since we're here". That
  domain has an owner; route. Duplicate findings under two labels is the failure
  mode this skill exists to kill.
- Reporting a finding with no failure story. "Non-standard" is not a finding;
  name who gets hurt, doing what, or drop it.
- Calling something missing without checking the layer above (shared middleware,
  framework default, DB constraint). The classic phantom.
- Auto-editing a public API shape, scriptable flag, or migration because the fix
  is "obviously right". Consumers script against the current shape; inbox it.
- Ranking by rubric-item name instead of impact × likelihood. A data-corrupting
  write path outranks twenty naming nits.
- A broad sweep that reports specialist domains itself instead of dispatching.
  "all" widens the routing, it doesn't suspend it.
- Padding the report with style nits to look thorough. Every finding must be
  worth the read; ten verified beats forty plausible.
- Banking a fix with no regression test where one could fail on the old code.

## References

- [RUBRICS.md](references/RUBRICS.md): the four native-domain rubrics expanded
  (API design, CLI ergonomics, data integrity, general code quality), each tuned
  to the pack's reference stack (Next.js/Supabase/Stripe SaaS, Rust CLIs, Electron), with what
  to grep for and the failure-story question that decides whether it's real.
