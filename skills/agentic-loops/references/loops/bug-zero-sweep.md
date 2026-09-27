---
name: bug-zero-sweep
goal: Drive the codebase's real bugs to zero, exhaustively
cadence: on-demand
run_via: /loop
routes_to: [bug-zero, no-slop-bug-hunter, bug-sweep]
stop_condition: Two consecutive full sweeps surface zero new *verified* bugs
verification_gate: Each bug confirmed real before fixing; each fix ships with a regression test under a green ratchet
---

# Bug-Zero Sweep

**Goal.** Not "find some bugs": drive them to zero, with every fix proven and
guarded so they don't come back.

**Each iteration.**
1. Run a finding pass (`bug-zero` orchestrating the bug-hunt skills; vary the lens each round: correctness, edges, concurrency, error paths).
2. **Verify each finding is real** before touching code: reproduce it or trace it to a concrete failure. Discard plausible-but-unconfirmed ones.
3. Fix one verified bug. Add a regression test that fails before the fix and passes after.
4. Run the green ratchet (offload the heavy suite to a build runner if you have one). Commit the fix + test together.

**Stop when** two consecutive full sweeps come up dry (no new verified bugs). Silence on one round isn't enough; the tail hides here.

**Green gate (every iteration).** Regression test added per fix; `build + lint + tests` green and never regressing; no fix lands without its proof.

## Copy-paste prompt
```
/loop Drive this codebase's bugs to zero. Each pass: run a bug-finding sweep (vary
the lens: correctness, edge cases, concurrency, error paths). Before fixing
anything, verify each finding is actually real by reproducing or tracing it; discard
unconfirmed ones. Fix one verified bug at a time, each with a regression test that
fails before and passes after, and keep the build+lint+tests green. Stop only when
two consecutive full sweeps surface zero new verified bugs.
```

**Cadence / notes.** Run before a release or after inheriting a messy codebase. For scale, fan the finders out across parallel agents (`swarm-run`) and keep verification single-threaded. Pairs with `security-sweep` for the security lens and `test-max` for the coverage tail.
