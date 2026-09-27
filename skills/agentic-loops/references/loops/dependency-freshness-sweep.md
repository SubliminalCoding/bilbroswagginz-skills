---
name: dependency-freshness-sweep
goal: Keep important repos current without careless churn
cadence: monthly (per maintained repo)
run_via: /loop or /schedule (monthly)
routes_to: [ship-it, reality-check]
stop_condition: No routine dependency update remains; only intentionally deferred risky upgrades, each with a logged reason
verification_gate: Tests pass AND the lockfile diff matches exactly the intended upgrade set (no surprise transitive bumps)
---

# Dependency Freshness Sweep

**Goal.** A broad repo surface (apps, services, and libraries across several
ecosystems) drifts fast. Controlled monthly freshness prevents painful surprise
upgrades and security drift, without the careless mass-bump that breaks a working
build.

**Each iteration.**
1. Check outdated dependencies for the repo; classify each as low-risk (patch/minor, well-tested) or risky (major, framework, native).
2. Upgrade the low-risk set in a batch; isolate each risky upgrade to its own branch/bead with a migration note.
3. Run the green ratchet (offload the heavy suite to a build runner if you have one); confirm the lockfile diff matches the intended set exactly.

**Stop when** no routine update remains, only deliberately deferred risky upgrades, each with a logged reason.

**Verification gate.** Tests pass AND the lockfile diff matches exactly the intended upgrade set. A surprise transitive bump or a red suite is FAILED; the loop never lands churn it didn't mean to.

## Copy-paste prompt
```
/loop Keep this repo's dependencies fresh without churn. Each pass: list outdated
deps and classify low-risk (patch/minor) vs risky (major/framework/native); batch-
upgrade the low-risk set and isolate each risky upgrade to its own branch+bead with a
migration note; run build+lint+tests and confirm the lockfile diff matches exactly
the intended upgrade set. A surprise transitive bump or a red suite is FAILED. Stop
when only intentionally-deferred risky upgrades remain, each with a logged reason.
```

**Cadence / notes.** Monthly per maintained repo. Pairs with `ship-it` and your release gate for the products you ship.
