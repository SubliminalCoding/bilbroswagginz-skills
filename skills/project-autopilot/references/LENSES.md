# Diagnostic Lenses

Each lens asks one narrow question against the Intent Statement. Run only the
lenses relevant to the project type. Relevance-filter FIRST, then diagnose.
A lens that doesn't apply emits nothing; it does not invent findings to feel useful.

## Lens output contract

Every lens emits zero or more gap records:

```
- gap: <one-line description>
  lens: <lens name>
  evidence: <file:line or observed behavior; no evidence, no gap>
  impact_on_intent: high | medium | low   # measured against the Intent Statement, not generic best practice
  effort: small | medium | large
  route: <skill from SKILL-ROUTING-MAP.md>
  mode: auto | ask-user
```

**impact_on_intent is the only score that matters.** A missing test suite is HIGH
for a billing service and LOW for a weekend toy. The Intent Statement decides.

## The lenses

| Lens | Key question | Relevant when | Irrelevant when |
|---|---|---|---|
| **correctness** | Does it actually work, end to end, on the happy path AND the edges? | always | never; this lens always runs |
| **performance** | Is anything slow enough that a user/operator would notice? | user-facing latency, hot loops, costs scale with load | static sites, one-shot scripts |
| **ux** | Would a first-time user get confused, stuck, or lost? | anything with a human interface (web, mobile, TUI, CLI) | pure libraries, daemons |
| **security** | Can someone exploit, leak, or bypass this? | network-facing, auth, payments, user data, secrets | local-only toys with no data |
| **architecture** | Is complexity proportionate? Will the next change be easy? | codebase still growing | frozen/maintenance-only |
| **testing** | Would a regression here be caught before the operator's users see it? | anything the operator ships to others | throwaway experiments |
| **copy** | Does the text (UI, README, errors) sound human and land clearly? | anything public-facing | internal-only tools |
| **operability** | Can it be deployed, observed, and recovered without archaeology? | services, anything with uptime expectations | CLIs, libraries |
| **distribution** | Can the intended user find it, try it, and pay for it? | commercial projects | fun side projects (per Intent Statement) |

## Lens → vertical handoffs

When a lens surfaces not one gap but a whole CLASS of them, route to that lens's
exhaustive vertical instead of grinding piecemeal fixes: **correctness** →
`bug-zero` · **performance** → `perf-max` · **ux** → `ux-maximizer` · **testing** →
`test-max` (the suite exists but its green can't be trusted: weak assertions,
flakes, line-coverage theater; late-stage, like the perf pass). Two cross-cutting
specialists sharpen individual findings before they route: a correctness gap that
can't be REPRODUCED goes to `trace-it` first (capture the bad state before any fixer
runs), and an architecture gap whose finding is disproportionate complexity goes to
`simplify-proof` (behavior-locked, measured reduction). Per-row routes and auto vs
ask-user stay in SKILL-ROUTING-MAP.md.

## Relevance filter procedure

1. Read the Intent Statement.
2. Classify the project: webapp / mobile / CLI / TUI / library / service / content.
3. Mark each lens relevant/irrelevant from the table; record the chosen set in the
   run log so a later tick doesn't silently re-litigate it.
4. The **distribution** lens is ask-user-heavy by nature: it mostly produces
   decision-inbox items, not auto work. Run it LAST and at most once per
   convergence cycle, not every tick.

## Shipped-reality vs code-reality (added after a pilot project's reality check, 2026-06-05)

Every lens above looks AT THE CODE. None of them asks whether the code has ever
run for a real user. The pilot found code gaps but missed that the
project's headline promise had **never
happened once**: empty git remote, no deploy, no real PR, every "it works" test
running against mocks. 50 beads closed, core value proposition at zero.

So before diagnosis, run a **shipped-reality probe** and record it in the run log:

```
- remote:   git remote -v            → is there anywhere to push?
- deploy:   is it actually running where its intended user can reach it?
- usage:    has anyone but the operator ever used it? (logs, analytics, a real PR/merge)
- e2e-real: does the headline promise have an UNMOCKED end-to-end proof?
```

Rule: a vision goal whose only proof is a mocked test is **STUB at shipped level**,
not WORKING, no matter how green the suite. The biggest gaps are almost never in
the code; they're in the space between the code and a real user. These gaps are
usually **ask-user** (create a public repo, deploy, authorize a domain). Surface
them at the TOP of the inbox, because closing the loop once is worth more than any
amount of localhost polish. Polishing a demo nobody has visited is the failure
mode this probe exists to catch.

## Re-verify before work (added after the first pilot, 2026-06-05)

Diagnosis agents embellish: in the first pilot ~3 of 20 findings were wrong,
self-retracted, or cited nonexistent endpoints. Before claiming a bead, the
executor MUST re-verify the gap's evidence in the actual code (open the cited
file:line, confirm the behavior). A finding that fails re-verification closes
its bead as false-positive with the evidence noted. That is completed work,
not skipped work.

## Anti-patterns

- Running all lenses on every project (noise; the relevance filter is the point)
- Gaps without file:line or reproducible-behavior evidence (hallucinated work)
- Scoring against "best practice" instead of the Intent Statement (gold-plating
  a fun side project, or under-investing in a commercial one)
- A lens re-reporting a gap that already has an open bead (duplicate work; check
  `br list` before emitting)
