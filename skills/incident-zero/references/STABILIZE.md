# Stabilize, Prove-Red, Prevent

Companion to incident-zero. The parts that need more room than the pipeline table:
the mitigation ladder, what to grab before you destroy it, how to meet the bug for
the fail-first gate, and where each incident class routes for prevention.

## Part 1: The mitigation ladder (most reversible first)

Work DOWN. Skipping a rung while a higher one is untried needs a reason written
down; skipping to an irreversible rung needs the operator.

| Rung | Lever | Undo | Notes |
|---|---|---|---|
| 1 | Revert the suspect deploy | redeploy | First when the incident correlates with a release. Check deploy history FIRST, always; "what changed" solves half of all incidents at rung 1. |
| 2 | Feature flag off | flip back | First when the change shipped behind a flag. Record WHICH flag, or Phase 5 can't unwind it. |
| 3 | Shed load / rate-limit / disable the non-critical path | remove the limit | Degrade on purpose beats down by surprise. |
| 4 | Scale out / restart | scale in | Restart DESTROYS evidence: capture first (Part 2). A restart that "fixes" it has hidden a leak/race, not fixed it; the incident continues at Phase 1. |
| 5 | Data surgery (fix rows, replay queue, clear poison message) | often none | BUTTON: ask-user, with the exact statement/command and its blast radius. Snapshot the bad data before touching it: it's Phase 1's best evidence. |
| 6 | Forced failover / region shift / DNS | slow, sometimes none | BUTTON: ask-user. |

Machine-level fires (load average, swap thrash, disk full, runaway process):
run system-performance-remediation (if installed; otherwise standard host triage)
as the rung-3/4 equivalent. On a shared, memory-constrained box, a memory fire
usually means a bare launch escaped memguard: kill by cgroup, then note the
missing wrap as a Phase 4 prevention item.

Every mitigation applied gets one line in the incident record AT THE TIME:
lever, when, undo. This list is Phase 5's unwind checklist.

## Part 2: Evidence-preservation checklist (before each destructive step)

Thirty seconds each; grab what the next lever will destroy:

- **Before restart:** process state: stack dump (`kill -3` / py-spy / gdb attach),
  `/proc/<pid>` status+fds, a core if it's wedged; recent logs if they rotate.
- **Before revert/redeploy:** the running SHA + config/env as deployed (Phase 3
  needs to check out exactly this to prove the test red).
- **Before data surgery:** snapshot/SELECT-INTO the bad rows; copy the poison
  message, don't just ack it.
- **Before flag flip:** current flag state and who/when it last changed.
- **Always:** timestamps of first alert, first user report, each mitigation.
  The timeline is reconstructable now and archaeology later.

## Part 3: The fail-first protocol (Phase 3's gate, mechanically)

The trap: Phase 0's mitigation usually removed the bug from HEAD, so a new test
runs green everywhere and proves nothing. Meet the actual bug:

1. Reconstruct the incident condition in a sandbox: check out the preserved
   pre-mitigation SHA, or flip the flag ON, or load the snapshotted bad data,
   whichever Phase 0 recorded.
2. Write the regression test asserting the INVARIANT the root-cause sentence
   names (not the stack trace's absence; the invariant).
3. RUN it there. Observe red. Paste the red output into the incident record.
   If it's green against the incident condition, the root-cause sentence is
   wrong: return to Phase 2, do not "fix" anyway.
4. Apply the minimal fix on HEAD (no-slop-bug-hunter owns the diff shape).
5. Run the test on HEAD+fix: green. Red→green is the artifact; suite green is
   the ratchet.

When the incident condition genuinely can't be reconstructed (data destroyed,
third-party state), say so in the record, pin the closest reproducible slice of
the invariant, and decision-inbox the residual risk. Never silently downgrade the
gate.

## Part 4: The prevention router (Phase 4)

Route the CLASS, not the instance. The regression test from Phase 3 already
guards the instance:

| Root cause shape | Prevention route |
|---|---|
| An invariant that plausibly breaks elsewhere (missing tenant scoping, unchecked null, unvalidated input) | bug-zero scoped to the class: sweep for siblings, each fix with its own regression test |
| The suite covered this path and asserted nothing / a flake masked the signal | test-max on the subsystem (mutation-proof the suite that lied; flake-zero the masker) |
| Detectable state preceded user impact | promote the decisive trace-it probe into a real monitor + alert; TEST-FIRE it (trigger the condition in staging, see the page) |
| A migration/cutover without parity or rollback rehearsal | adopt migrate-proof as the standing gate for that system's changes |
| An ops/method lesson that generalizes across projects | skillify: the fleet learns it once |
| Human-process hole (no deploy history, no flags, no staging) | decision-inbox for the operator: process fixes are the operator's call, with a recommended option |

One route minimum; two when the incident was severe (e.g. the sweep AND the
alert). The artifact must be verifiable: a sweep has findings-or-dry output, an
alert has a test-fire, a skill has a registered entry. "We'll be more careful"
is not an artifact.

## Part 5: The incident record (Phase 5's close-out)

One file, `<project>/.incidents/YYYY-MM-DD-<slug>.md`, four sections, all
required to close: **Timeline** (from Part 2's stamps) · **Root cause** (the one
sentence + the trace capture) · **Regression test** (path + the pasted red run +
the green run) · **Prevention** (route(s) taken + verification evidence), plus
the mitigation-unwind checklist, each item ticked or explicitly kept-with-reason.
