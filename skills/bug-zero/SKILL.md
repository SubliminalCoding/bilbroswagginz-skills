---
name: bug-zero
description: >-
  Drive a codebase's bugs to zero, exhaustively, by orchestrating every bug-finding
  skill you have installed, then verifying each finding is real before fixing, fixing under a
  green ratchet with a regression test per fix, confirming the fix by running it, and
  rescanning until the sweeps come up dry. Use when "find and fix all the bugs",
  "harden this", "pre-release bug pass", "make this bulletproof", post-incident
  cleanup, or taming an inherited messy codebase.
---

# Bug Zero

Point it at code and drive the bugs to zero. It doesn't reinvent bug-finding; it
**conducts the bug-finders you already have**, and adds the trust layer they lack:
nothing gets fixed until it's verified real, no fix is banked until it's proven to
work and guarded by a test, and the loop stops only when fresh sweeps come up empty.

> **Core Insight:** You already have great bug *finders*. What's missing is the
> conductor that makes them *trustworthy*. The hard part of fixing code with AI
> isn't finding bugs. It's not hallucinating them, proving a fix actually fixed it
> without breaking anything else, and knowing when you're done. Like its siblings
> **project-autopilot** and **ux-maximizer**, this is a *successive-prompter* with a
> **green ratchet**: it runs almost no logic of its own; it sequences existing
> skills and wraps them in verification.

## What bug-zero adds (the only new logic)

1. **Multi-modal fan-out:** runs the finders together; each catches a bug class the
   others miss (general / concurrency / fakes / diff / live-process / security).
2. **Adversarial verification:** the smart model re-checks EVERY finding against the
   real code before a single fix. Finders over-report; phantom bugs die here.
3. **Green-ratchet fixing + a regression test per fix:** a "fix" that reds any test
   reverts; every real fix ships with a test that would have caught the bug.
4. **Loop-until-dry convergence:** stops on saturation (diminishing real bugs per
   token), not guesswork and not literal zero.

## Preconditions / guards

1. **Baseline must be green:** the ratchet needs a known-good starting point. If the
   suite is ALREADY red at launch, that's an escalation: triage to a green baseline
   or get the operator's call on what "green" means here *before* looping. (You can't ratchet
   from a red baseline.)
2. **Checkpoint first:** recoverable starting commit before any fix.
3. **Never weaken tests to reach green.** Deleting/skipping assertions to force a
   pass is forbidden; that hides bugs, it doesn't fix them.
4. **Get a repro first.** A symptom that can't be reproduced can't be verified real
   or proven fixed. Route it to trace-it FIRST (instrument, capture the bad state,
   return with a located cause); the capture then enters Phase 2 like any other
   finding. And a LIVE production fire is not a bug pass at all; that's the sibling
   incident-zero (stabilize before understanding); it hands the post-incident
   cleanup back here once users are safe.

## The Pipeline (staged: issue each step only when the prior finishes)

```
Phase 0  ORIENT     Read the code + understand intent; establish + checkpoint a
                    green baseline (see guard #1).
Phase 1  SWEEP      Fan out the finders IN PARALLEL, each a different bug class.
                    Dedupe the union of findings. (Roster: references/ROUTING.md.)
Phase 2  VERIFY     Smart-model adversarially re-checks EVERY finding against the
                    actual code. Doubtful ones → multi-model-triangulation /
                    lean-formal-feedback-loop (if installed). Phantom bugs die HERE, pre-fix.
Phase 3  FIX        Fix the survivors one batch at a time (≤5) under the green
                    ratchet, each paired with a regression test that would have
                    caught it. Fix reds a test → revert that batch, move on.
Phase 4  PROVE      Reproduce that the bug is actually GONE (verify skill / run it),
                    not just that tests pass. A fix you didn't watch work isn't one.
Phase 5  RESCAN     Re-sweep. Stop when K=2 consecutive multi-modal passes find
                    nothing real (saturation). Else → back to Phase 2.
```

## Skills it orchestrates

bug-zero is built on the finders you have installed; it adds sequencing + verification,
not finders. The non-pack finders below are optional: use each if installed;
otherwise the smart model runs that bug-class check itself. Full bug-class → skill map and the staged prompts are in
[references/ROUTING.md](references/ROUTING.md). In brief: **repro** with trace-it
(the un-reproducible symptom is instrumented into evidence before any finder runs);
**find** with ubs / bug-sweep / deadlock-finder-and-fixer / mock-code-finder /
code-review / gdb-for-debugging / security-sweep; **verify** with multi-model-triangulation
/ lean-formal-feedback-loop; **prove** with verify / the testing-* family; **clean up**
with simplify / deps-update when the bug was really complexity or a stale dep.

## Shared mechanisms (inherited from project-autopilot)

Green ratchet, decision inbox (judgment calls queue for the operator), saturation convergence,
and capability-aware execution (solo by default; ntm swarm optional when installed) work as in project-autopilot; see
that skill's CONVERGENCE.md / SAFETY-RAILS.md / EXECUTION.md rather than duplicating.
**Integration:** project-autopilot's correctness lens routes a project here for the
exhaustive treatment, the same way the UX lens routes to ux-maximizer.

## THE EXACT PROMPT

```
/bug-zero
Target: <project path or file/module>
Scope: <whole project | a subsystem | the current diff>
Mode: solo (default) | swarm (optional; needs ntm installed)
```

## Anti-Patterns

- Fixing a finding before verifying it's real: the cardinal sin; finders over-report.
- Running ONE finder. Multi-modal fan-out is the point (each catches what others miss).
- Claiming "fixed" without reproducing the bug is actually gone.
- Banking a fix with no regression test: it comes back.
- Deleting / weakening / skipping tests to reach green: that hides bugs.
- Ratcheting from an already-red baseline (escalate first; you have no green to hold).
- Grinding past saturation, or stopping early at literal zero instead of K dry sweeps.

## References

- [ROUTING.md](references/ROUTING.md): bug-class → finder map, verify/prove/cleanup rosters, staged prompts
