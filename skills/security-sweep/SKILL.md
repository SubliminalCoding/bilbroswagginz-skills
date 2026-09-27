---
name: security-sweep
description: >-
  Audit a SaaS app for the security classes that actually sink companies:
  payment/billing bypass, webhook integrity (signature + replay), auth gaps
  (authn/authz, session/JWT/cookie), row-level-security and multi-tenant data
  isolation, secrets exposure, and IDOR/broken access control. Verifies each
  finding is real before reporting, ranks by blast radius, and ships a
  concrete fix per finding. Use when "security audit", "billing security",
  "is this safe to ship", webhook/auth/RLS review, pre-launch hardening, or when
  bug-zero / ship-it / project-autopilot route the security class here.
---

# Security Sweep

Point it at a SaaS app and it finds the holes that cost money or leak tenants,
not a generic lint pass. It concentrates fire on the handful of classes with real
blast radius (someone pays $0, someone reads another tenant's data, a key leaks),
proves each one is actually exploitable before it says a word, ranks by damage,
and hands back a fix per finding. Low-risk fixes land under a green ratchet;
anything touching a live payment or auth path is queued for the user, never auto-edited.

> **Core Insight:** A security finding is worthless until it's *proven reachable*.
> Scanners and smart models both over-report: they flag the shape of a bug without
> tracing whether an unauthenticated request can actually get there. The hard part
> isn't listing OWASP categories; it's building the exploit path in your head
> (who, unauthenticated or cross-tenant, hits which route, bypassing which check)
> and rejecting the plausible-but-wrong noise. Report only what you'd bet on. And
> because the highest-value classes (billing and auth) are also the ones where a
> wrong "fix" causes an outage or a revenue leak, those changes are **proposed, not
> applied**: findings auto, payment/auth-critical edits to the decision inbox.

## Preconditions / guards

1. **Know the stack before you grep:** Next.js route handlers vs. Edge middleware,
   Supabase RLS vs. service-role keys, Stripe vs. RevenueCat webhooks each fail
   differently. Read how auth, billing, and the data layer are wired *first*; a
   finding written against the wrong mental model is a phantom.
2. **Green ratchet on any applied fix:** A fix that reds a test reverts. Never
   weaken a test to pass.
3. **Payment-flow and auth-critical changes are ask-user:** Produce the finding and
   the recommended fix, but write the change to `<project>/.autopilot/DECISIONS.md`
   (recommended option first) and DO NOT auto-edit a live billing or session path.
   A bad auto-fix here is an outage or a free-money bug.
4. **Read-only against real services:** Never fire test webhooks at prod, never
   exercise a live charge path to "confirm". Reason the exploit; reproduce only in a
   test-mode/local sandbox.

## The audit classes (where the fire goes)

Full checklist with Next.js/Supabase/Stripe-specific failure modes in
[references/CLASSES.md](references/CLASSES.md). The seven:

1. **Payment / billing bypass:** entitlement granted without a settled charge;
   client-trusted price/quantity/plan; webhook-before-payment ordering; refund/
   downgrade that never revokes access; RevenueCat receipt trusted unsigned.
2. **Webhook integrity:** missing/incorrect signature verification (Stripe
   `constructEvent`, RevenueCat auth header), no replay/idempotency guard, raw-body
   mismatch from a framework that already parsed the body.
3. **Auth gaps:** authn vs authz confusion, missing server-side check behind a
   hidden UI control, JWT not verified / `alg:none` / wrong audience, session fixation,
   cookies missing `HttpOnly`/`Secure`/`SameSite`, `getSession()` trusted where
   `getUser()`/`getClaims()` is required.
4. **RLS / multi-tenant isolation:** RLS disabled or `USING (true)`, a service-role
   key reachable from a user-facing path (bypasses RLS entirely), tenant_id taken from
   the request instead of the verified session, a query that forgets the tenant filter.
5. **Secrets exposure:** keys in source / committed `.env` / logs, a server secret
   prefixed `NEXT_PUBLIC_` and shipped in the client bundle, service-role key in
   browser code.
6. **IDOR / broken access control:** object id from the URL with no ownership check,
   sequential ids, mass-assignment letting a request set `role`/`tenant_id`/`is_admin`.
7. **Input → injection surface** (secondary): only where it crosses into the above
   (SQL via raw query bypassing RLS, SSRF reaching internal billing APIs).

## The Pipeline (staged: issue each phase only when the prior finishes)

```
Phase 0  MAP        Read how auth, billing, the data layer, and webhooks are wired
                    (guard #1). List every trust boundary: each route handler, each
                    webhook endpoint, each Supabase query path, each entitlement
                    gate. This map is the attack surface.
Phase 1  SWEEP      Walk the seven classes against the map. For each candidate,
                    write the EXPLOIT PATH: who (unauth / other tenant / lower role)
                    hits what, bypassing which absent check, to what effect. No
                    exploit path = not a finding yet.
Phase 2  VERIFY     Adversarially re-check EVERY candidate against the real code:
                    open the cited file/line, confirm the missing check is actually
                    missing (not enforced one layer up in middleware / RLS / a
                    guard). Default to "not a finding" if you can't trace the path.
                    Doubtful + high-stakes → multi-model-triangulation if installed,
                    else a second independent trace. Phantoms die
                    HERE, before anything is written or changed.
Phase 3  RANK       Order survivors by blast radius × reachability: unauth remote
                    money/tenant-leak first, then authed cross-tenant, then
                    same-tenant escalation, then info-leak. Severity is damage ×
                    ease, not category name.
Phase 4  REMEDIATE  Split by risk:
                    • Low-risk (add missing authz check on a non-billing route, set
                      a cookie flag, move a secret, add an ownership filter) → fix
                      under the green ratchet, each with a regression test that fails
                      on the vulnerable code and passes on the fix.
                    • Payment-flow / auth-critical / RLS-policy change → DO NOT EDIT.
                      Write finding + recommended fix to .autopilot/DECISIONS.md.
Phase 5  PROVE      For each applied fix, prove the exploit is now blocked (test-mode
                    repro / the regression test), not just that the suite is green.
                    Re-sweep the touched surface; a fix often exposes an adjacent gap.
```

## Output (what orchestrators get back)

- **Ranked verified findings:** each with: class, severity (blast radius ×
  reachability), the exploit path, cited file:line, and the concrete fix.
- **Applied fixes:** low-risk only, green-ratcheted, each with its regression test.
- **`<project>/.autopilot/DECISIONS.md`:** appended: every payment-flow / auth-
  critical / RLS-policy change, recommended option first, for the user to approve. Same
  inbox project-autopilot and idea-forge drain: one inbox, not three.

## Contract (what orchestrators rely on)

- **bug-zero** routes its security class (auth bypass, webhook integrity, RLS,
  secrets) here in the SWEEP phase; this skill returns verified findings into
  bug-zero's own verify/fix loop.
- **project-autopilot** routes "Billing/payment/webhook security" here. Findings
  produced AUTO, payment-flow CHANGES ask-user via the decision inbox.
- **ship-it** calls it as a pre-ship hardening gate.
- **Most specific skill wins:** chosen over generic audit-lens for any billing /
  auth / webhook / multi-tenant code.

## Skills it composes with

These are optional; use any that are installed, otherwise the provider's own docs
cover the same ground. Verify doubtful findings with **multi-model-triangulation**;
for billing flows pair with **saas-billing-patterns-for-stripe-and-paypal** and
**stripe-checkout** to know the correct shape before calling a deviation a bug;
**supabase** for RLS/auth specifics; prove fixes with
**testing-real-service-e2e-no-mocks** (Stripe test mode, real-DB tenant-isolation
tests). It adds the security lens + exploit-path verification;
these supply the domain detail.

## THE EXACT PROMPT

```
/security-sweep
Target: <project path or subsystem (e.g. the billing + webhook routes)>
Scope:  <whole app | billing/webhooks | auth/session | tenant isolation>
Stack:  <e.g. Next.js + Supabase RLS + Stripe webhooks>   (optional; it will infer)
```

## Anti-Patterns

- Reporting a finding with no exploit path. "Looks risky" is not a vulnerability;
  trace who reaches it bypassing which check, or drop it.
- Calling a missing check a bug without confirming it isn't enforced one layer up
  (middleware, RLS policy, a shared guard). The classic phantom.
- Auto-editing a live payment or session path. That's an outage / revenue-leak risk;
  it goes to the decision inbox, full stop.
- Firing real webhooks or live charges at prod to "confirm". Repro in test mode only.
- Ranking by OWASP category name instead of blast radius × reachability. An unauth
  remote tenant-leak outranks a same-tenant self-XSS every time.
- Banking a fix with no regression test that fails on the vulnerable code.
- Trusting `getSession()` / client-supplied price / client-supplied tenant_id and not
  flagging it because "the UI never sends that". The attacker isn't using your UI.
- One generic scan instead of walking the seven classes against the actual trust map.

## References

- [CLASSES.md](references/CLASSES.md): the seven audit classes expanded, each with
  Next.js / Supabase / Stripe / RevenueCat failure modes, what to grep for, and the
  exploit question that decides whether it's a real finding.
