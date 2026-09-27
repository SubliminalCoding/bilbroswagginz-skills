# Audit Lens: Native-Domain Rubrics

The four domains audit-lens owns (no specialist in the pack). Each rubric:
what to check, stack-specific failure modes, what to grep for, and the
**failure-story question**. If a candidate finding can't answer it, it's not a
finding (Phase 2/3 of the pipeline).

The routing table for everything else lives in SKILL.md; do not duplicate it
here. If a domain below ever gets a dedicated specialist, the table wins and
this rubric retires for that domain.

---

## 1. API design

**Failure-story question:** which consumer (frontend, mobile app, webhook
receiver, third-party integrator, future-you in six months) does what against
this surface and gets surprised, broken, or stuck?

Check:

- **Consistency of shape**: same resource serialized differently across
  routes; `userId` here, `user_id` there; one route returns the object, its
  sibling returns `{ data: object }`. Consumers write one parser; every
  divergence is a bug they'll ship.
- **Error envelope**: errors as 200-with-`{error}`, raw framework stack traces
  in the body, status codes that lie (404 for unauthorized, 500 for validation).
  One documented envelope, correct codes, no internals leaked.
- **Validation at the boundary**: request bodies trusted into the handler
  (`await req.json()` straight into a query) instead of parsed through a schema
  (zod or equivalent). Distinct from *security* injection (that routes to
  security-sweep); this lens is about garbage-in producing 500s and corrupt
  state instead of a 400.
- **Verb/idempotency semantics**: GETs that mutate, POSTs that should be
  idempotent PUTs, no idempotency key on anything a client will retry (anything
  that creates a charge-adjacent or unique resource).
- **Pagination / unbounded lists**: list endpoints with no limit; works in
  dev with 50 rows, times out in prod with 50k.
- **Versioning & breaking-change posture**: renamed/removed fields with no
  deprecation path; if there are external consumers, any shape change is an
  inbox item, never an auto-fix.
- **Contract truthfulness**: the OpenAPI doc / README / TS types describe a
  shape the handler no longer returns.

Stack notes (Next.js/Supabase):

- Route handlers vs. server actions doing the same job differently: pick the
  project's convention and flag drift, don't impose one.
- Supabase client calls from the browser ARE API surface: RLS-shaped, schema
  changes break them just like route changes. (Whether RLS is *correct* →
  security-sweep; whether the shape is *coherent* → here.)
- `NextResponse.json(err)` passing a caught exception object straight through:
  the classic stack-trace leak that's an API-design smell before it's a
  security one.

Grep starters: `route.ts|route.js` inventory; `req.json()` without a nearby
`parse|safeParse`; `NextResponse.json` second-arg status audit; `catch` blocks
that return the error object.

## 2. CLI ergonomics

**Failure-story question:** a scripter pipes this into `jq`/`xargs`/CI, or a
new user runs it cold: where does it lie to them, block them, or break their
pipe?

Check:

- **stdout/stderr discipline**: data on stdout, diagnostics on stderr, never
  mixed. Progress bars and log lines on stdout poison every pipe.
- **Exit codes**: non-zero on failure, zero only on success; distinct codes
  for distinct failure classes if anything scripts against it; never `Ok(())`
  after printing an error.
- **`--json` (or equivalent) on anything inspectable**: humans get tables,
  scripts get JSON; if agents will drive the tool (they will, on an agent-driven stack),
  machine output is a requirement, not polish.
- **Help text that teaches**: every subcommand has a one-line description and
  at least one example; `--help` is the docs most users ever read.
- **Flag conventions**: long flags for everything, short flags only for the
  hot few; consistent verbs across subcommands (`list/show/create/delete`, not
  `list/view/new/rm`); destructive ops need `--force` or a confirm.
- **Respects the contract of the environment**: `NO_COLOR`, non-TTY detection
  (no ANSI into a pipe), config file + env var + flag precedence documented and
  consistent.
- **Error messages that name the fix**: "config not found at ~/.foo/config.toml
  (run `foo init`)" not "Error: NotFound".
- **Startup cost**: a CLI invoked per-item in a loop must not pay a 2s init
  each call. (Measuring how slow → hotspot-scout; *that it matters* is
  ergonomics.)

Stack notes (Rust CLIs):

- clap derive gives help/version/typo-suggestions for free; hand-rolled arg
  parsing next to a clap dependency is a finding.
- `unwrap()`/`expect()` on user-reachable paths = panic + backtrace as the UX;
  user-facing errors go through `anyhow`/`miette` with context.
- `println!` for status output in a tool with a `--json` flag: grep for
  `println!` outside the render layer.
- br/bv (if installed) are the house reference for agent-friendly CLI shape (JSONL out,
  composable subcommands); compare against them, or against any agent-friendly CLI
  you already use, not against an abstract ideal.

Grep starters: `eprintln!|println!` distribution; `std::process::exit`
codes; `unwrap()|expect(` outside tests; clap derive presence vs. manual
`args()` loops.

## 3. Data integrity / data-model

**Failure-story question:** which write path, crash, retry, or migration leaves
the data wrong, and would anyone notice before a customer does?

Check:

- **Constraints live in the database**: uniqueness, FKs, NOT NULL, CHECK
  enforced in Postgres, not only in app code; every app-only invariant is one
  concurrent request away from violation.
- **Orphans and cascades**: FKs without a deliberate `ON DELETE` choice;
  deletes that strand child rows; soft-delete flags that half the queries
  forget to filter.
- **Migrations**: schema drift (prod ≠ committed migrations); destructive
  migrations (column drop/rename) with no backfill or expand-contract plan;
  migrations that can't run twice. Any migration is inbox-tier, never auto-run.
- **Transactional boundaries**: multi-write operations (create org + owner +
  subscription row) not wrapped in a transaction; partial failure leaves a
  half-created entity.
- **Idempotency of retried writes**: webhook handlers and queue consumers that
  insert without an idempotency key/unique constraint; Stripe WILL redeliver.
  (Signature verification → security-sweep; duplicate-row corruption → here.)
- **Money and time**: currency as float anywhere (cents-as-integer or numeric,
  full stop); timestamps without timezone; truncation where rounding rules
  matter.
- **Status-field state machines**: `status` strings mutated freely with no
  enum/CHECK and no legal-transition enforcement; the source of "how did it get
  refunded before it was paid".
- **Derived data**: counters/denormalized rollups with no reconciliation path;
  caches of DB state with no invalidation story.

Stack notes (Supabase/Stripe; Electron):

- Webhook-driven entitlement rows are the highest-stakes table in a SaaS:
  unique constraint on the event id, idempotent upsert, transaction around
  grant + record.
- Supabase migrations in `supabase/migrations/` vs. dashboard-edited schema:
  diff them; dashboard drift is the default failure mode.
- Electron local stores (SQLite/JSON) sync against the server copy: which is
  authoritative, and what happens on conflict, must be answerable.

Grep starters: migration directory vs. live schema diff; `float|real` on money
columns; insert paths in webhook/queue handlers without `on conflict`/unique
key; multi-statement writes without `transaction|begin`.

## 4. General code quality

**Failure-story question:** the next maintainer (probably an agent in a swarm)
changes something here: what do they break because the code hid it from them?

This lens is structural. Specific bugs → bug-zero; slop in docs/copy →
unslop; perf → hotspot-scout. What's left, and what this rubric owns:

- **Dead and duplicated code**: unreferenced modules, the same logic
  hand-copied in three handlers (drifted into three slightly different
  behaviors; flag the drift, that's the cost), commented-out blocks kept "just
  in case".
- **Swallowed errors**: `catch {}` / `let _ =` on fallible ops / `.catch(() =>
  null)`; the audit isn't "add handling everywhere", it's "every swallow is a
  deliberate, commented choice".
- **Boundary discipline**: DB queries in UI components, business logic in
  route handlers that also format HTTP, one 800-line module that is the app.
  Findings here are inbox-tier refactors, not auto-fixes.
- **Config and magic values**: the same URL/limit/timeout hardcoded in four
  places; env vars read deep in call stacks instead of validated once at
  startup.
- **Test health**: not coverage theater. Do tests exist where the failure
  stories above live (the write paths, the parsers, the money math)? Skipped/
  `.only`'d tests committed; tests that assert nothing.
- **Dependency hygiene**: unused deps, two libraries doing the same job, a
  fork/patch with no note saying why.
- **Drift between doc and code**: README/AGENTS.md describing commands,
  flags, or flows that no longer exist; for agent-driven repos this is the
  highest-leverage quality fix there is.

Stack notes:

- Next.js: client/server boundary leaks (`use client` on whole trees, server
  code imported into client bundles): structural, before it's a perf or
  security issue.
- Rust: `clippy --all-targets -- -D warnings` as the floor; `#[allow]` without
  a justifying comment is a finding; pervasive `clone()` to satisfy the borrow
  checker marks a design seam (flag the seam, don't micro-optimize; that's
  optimize-proof's job).
- Electron: main/renderer/preload separation: renderer reaching for Node APIs
  directly is structural rot (and a security finding security-sweep will also
  catch; report the structure here, route the exploit there).

Grep starters: `catch\s*\{\s*\}|catch (e) {}` and `let _ =` on `Result`;
`TODO|FIXME|HACK` inventory (age them via blame); `.skip|.only` in tests;
unreferenced exports.

---

## Broad sweep (`Domain: all`): merge rules

1. Dispatch specialist domains first (they're slower); run the four native
   rubrics while they execute.
2. One report, one ranking. Tag each finding with its producing skill.
   Cross-domain duplicates (Electron renderer/Node access, webhook
   idempotency) keep the **specialist's** version; the native finding merges
   into it as context.
3. Impact × likelihood ranks across domains: a security-sweep critical
   outranks every native finding; a data-corruption finding outranks any
   ergonomics nit. Do not quota per domain to look balanced.
4. One decision inbox. Specialists append their own items; audit-lens appends
   its native items; nothing is double-entered.
