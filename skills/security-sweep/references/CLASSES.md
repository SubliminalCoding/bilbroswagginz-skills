# Audit Classes: Security Sweep

The seven classes, expanded for the pack's reference stack (Next.js + Supabase RLS + Stripe /
RevenueCat). For each: the failure modes, what to grep for, and the **exploit
question** that decides whether a candidate is a real finding. A candidate with no
answerable exploit question is not a finding. Drop it in Phase 2.

Severity rule everywhere: **blast radius × reachability**. Unauthenticated + remote +
money/tenant-leak is the top of the stack; same-tenant self-inflicted is the bottom.

---

## 1. Payment / billing bypass

**Failure modes**
- Entitlement (Pro tier, credits, feature flag) granted on the client's word, or on a
  checkout *redirect* return, instead of on a settled webhook event.
- Price / quantity / plan id read from the request body and trusted: attacker sets
  amount to 0 or swaps a $99/yr price for $0.
- Webhook ordering: access granted on `checkout.session.completed` before
  `payment_intent.succeeded`, so a created-but-unpaid session unlocks access.
- Refund / dispute / subscription-cancelled / downgrade never *revokes* entitlement:
  user keeps Pro after charging back.
- RevenueCat: trusting the client SDK's `customerInfo` for entitlement instead of the
  server-validated receipt / webhook; trusting a sandbox receipt in prod.

**Grep / look**: `checkout.session`, `subscription`, `entitlement`, `tier`, `is_pro`,
`price`, `amount`, `quantity`, RevenueCat `customerInfo`, where the DB row that grants
access gets written.

**Exploit question**: Can a user obtain or keep paid access without a *settled,
server-verified* charge, by editing a request, replaying a redirect, or never
triggering the revoke path?

---

## 2. Webhook integrity

**Failure modes**
- No signature verification: endpoint trusts any POST. (Stripe: not calling
  `stripe.webhooks.constructEvent` with the signing secret; RevenueCat: not checking
  the `Authorization` header you configured.)
- Raw-body mismatch: Next.js / a body parser already parsed JSON, so the signature is
  computed over re-serialized bytes and either always fails (and someone "fixed" it by
  skipping verification) or is verified against the wrong bytes.
- No idempotency / replay guard: the same signed event replayed grants credits twice
  or re-runs a side effect. Need a processed-event-id table or idempotency key.
- Signing secret hardcoded or wrong (test secret in prod).

**Grep / look**: `constructEvent`, `webhook`, `signature`, `stripe-signature`,
`rawBody`, `bodyParser`, `export const config`, the webhook route handler's body access.

**Exploit question**: Can an attacker POST a forged or *replayed* event to the webhook
URL and have it accepted, granting entitlement, credits, or any state change?

---

## 3. Auth gaps (authn / authz, session / JWT / cookie)

**Failure modes**
- **Authz behind hidden UI**: the server route has no permission check; it relied on
  the button being hidden. Every mutating route handler needs a server-side check.
- **Supabase**: trusting `supabase.auth.getSession()` (reads unverified local JWT) for
  authorization where `getUser()` / `getClaims()` (server-verified) is required.
- JWT: signature not verified, `alg:none` accepted, wrong `aud`/`iss`, expired token
  accepted, secret weak/shared.
- Session: fixation (id not rotated on login), no server-side invalidation on logout,
  overly long lifetime, no rotation.
- Cookies: missing `HttpOnly`, `Secure`, or `SameSite`; auth token in `localStorage`
  (XSS-readable) instead of an HttpOnly cookie.

**Grep / look**: `getSession`, `getUser`, `getClaims`, `jwt`, `verify`, `cookies()`,
`Set-Cookie`, `httpOnly`, `sameSite`, route handlers under `app/api`, middleware.

**Exploit question**: Can a request without valid credentials (or with a lower-
privileged user's credentials) invoke this action or read this data?

---

## 4. RLS / multi-tenant isolation

The class that leaks one customer's data to another. Critical for multi-tenant apps
(e.g. a B2B SaaS where each customer business is a tenant).

**Failure modes**
- RLS disabled on a table that holds tenant data, or a policy of `USING (true)`.
- **Service-role key reachable from a user-facing path**: the service role *bypasses
  RLS entirely*, so any route that uses it must do its own tenant scoping. A leaked or
  client-side service-role key is total compromise.
- `tenant_id` / `org_id` taken from the request (body, query, header) instead of the
  verified session; attacker sets another tenant's id.
- A query that simply forgets the tenant filter, relying on RLS that isn't there.
- A new table/column added without a matching policy.

**Grep / look**: `service_role`, `SUPABASE_SERVICE_ROLE_KEY`, `createClient` (which
key?), `enable row level security`, `using (`, `tenant_id`, `org_id`, raw SQL,
`.from(` calls and whether they're scoped.

**Exploit question**: As tenant A, can I read or write tenant B's rows, by changing an
id, hitting a service-role-backed route, or because no policy / filter constrains me?

---

## 5. Secrets exposure

**Failure modes**
- Keys committed to source or in a checked-in `.env`.
- A server secret prefixed `NEXT_PUBLIC_`: Next.js inlines it into the **client
  bundle**, shipping it to every visitor. (Stripe secret key, service-role key,
  webhook secret must NEVER be `NEXT_PUBLIC_`.)
- Service-role key or Stripe secret key imported into a client component / browser code.
- Secrets logged (request bodies, headers, error dumps) to console / log drains.

**Grep / look**: `NEXT_PUBLIC_`, `SERVICE_ROLE`, `sk_live`, `sk_test`, `whsec_`,
`process.env` usage in `'use client'` files, `console.log` near secrets, `.env*` in git.

**Exploit question**: Can a secret that grants server power be read from the client
bundle, a public log, or the git history?

---

## 6. IDOR / broken access control

**Failure modes**
- Object id from the URL / body used to fetch without an ownership check
  (`/api/invoice/123` returns anyone's invoice).
- Sequential / guessable ids that make enumeration trivial.
- **Mass assignment**: spreading the request body into an update lets the attacker set
  `role`, `is_admin`, `tenant_id`, `plan`, `balance`.
- Function-level: admin-only route with no role gate.

**Grep / look**: `params.id`, `req.body` spread into `update`/`insert`, `...body`,
`role`, `is_admin`, route handlers that fetch by id, `.update(` with un-whitelisted fields.

**Exploit question**: By changing an id or adding a field to my request, can I access or
mutate an object I don't own, or grant myself a privilege?

---

## 7. Input → injection (secondary; only where it crosses into the above)

Not a general injection audit; flag only where it reaches a high-blast-radius class.

**Failure modes**
- Raw SQL built from input that bypasses RLS / parameterization.
- SSRF where a fetched URL reaches an internal billing / metadata endpoint.
- Command / template injection in a server path that touches secrets or money.

**Exploit question**: Does this untrusted input reach a sink that leaks a tenant, moves
money, or exposes a secret? If it's just reflected text with no such reach, it's not in
scope for this sweep.

---

## Verification discipline (applies to all seven)

Before any candidate becomes a reported finding:
1. Open the cited file:line. Confirm the missing check is *actually* missing, not
   enforced in middleware, an RLS policy, or a shared guard one layer up.
2. Trace the full path from an untrusted caller to the impact. If you can't, it's not a
   finding.
3. Default to "not a finding" under uncertainty. A phantom reported erodes trust in the
   whole sweep; a near-miss skipped gets caught next pass.
4. High-stakes + doubtful → multi-model-triangulation (if installed; otherwise a
   second independent trace) before committing.
