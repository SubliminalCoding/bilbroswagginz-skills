---
name: incident-zero
description: >-
  Take a production incident from "it's on fire" to "this class can't happen
  again": stabilize first with the most reversible lever (buttons for anything
  irreversible), gather evidence via trace-it, write a single-sentence root cause,
  fix under the green ratchet with a regression test proven to FAIL on the
  incident, and require prevention before the incident may close. Use when "prod
  is down", "something broke in production", "users are seeing errors", a paging
  alert, or a post-mortem that must end in prevention, not prose.
---

# Incident Zero

The "something broke in prod" entry point: the lifecycle conductor forge routes to
when the DETECT step finds a fire instead of a stage. Like its conductor siblings
it adds almost no logic of its own. It **sequences the specialists you already
have** (trace-it → no-slop-bug-hunter → test-max/bug-zero → skillify → ship-it) and
adds the two disciplines incident response always loses under adrenaline: the
order, and the exit bar.

> **Core Insight:** Incidents are mishandled in two opposite ways. Debugging while
> the fire burns (users pay by the minute for your curiosity), and closing when
> the fire is out (the class returns next month wearing a different stack trace).
> So the conductor forces both ends: **stabilize before you understand, and prevent
> before you close.** Mitigation and root-cause are different activities on
> different clocks. The moment users are safe, the emergency is over and every
> remaining step runs at engineering speed under engineering discipline (green
> ratchet, no-slop, buttons for anything irreversible).

## What incident-zero adds (the only new logic)

1. **Stabilize-first sequencing.** The most reversible lever is pulled first, evidence
   preserved on the way, understanding deferred until users are safe.
2. **The fail-first regression gate.** The regression test must be RUN against the
   incident code and OBSERVED red before the fix banks. A regression test that was
   never seen failing on the actual incident is a test of your theory, not of the
   incident.
3. **Mandatory prevention.** A fixed incident is an open incident. The CLASS gets a
   durable guard before close, or the incident stays open in the inbox.
4. **Ask-operator on every irreversible move under pressure:** data surgery, forced
   failover, destructive restarts, anything outward. Adrenaline is exactly when the
   button discipline matters most (ship-it's rule, inherited whole).

## Preconditions / guards

1. **Evidence preservation beats tidiness.** Before any restart/revert/scale action,
   snapshot what it will destroy (logs, process state, the bad rows, a core dump if
   it's a corpse). Thirty seconds of capture now saves the whole Phase 1.
2. **Reversible-first ladder is mandatory** (see the STABILIZE phase). Jumping to an
   irreversible lever while a reversible one is untried needs the operator's go.
3. **The green ratchet applies from Phase 3 on.** During STABILIZE, prod state is
   the checkpoint discipline (know the rollback of every mitigation); from FIX
   onward it's the normal ratchet: checkpoint, batch, revert-on-red.
4. **No fix work during Phases 0–2.** The urge to fix while stabilizing ships
   untested code into a fire; the urge to fix while tracing is trace-it's fix
   freeze. Both are forbidden here by inheritance.

## The Pipeline (staged: each phase's exit bar holds before the next begins)

```
Phase 0  STABILIZE   Stop the bleeding, most reversible lever first:
                     revert the suspect deploy → flip the flag off → shed/scale/
                     restart → data surgery [BUTTON — ask-user] → forced failover
                     [BUTTON]. Preserve evidence BEFORE each destructive step.
                     If the incident is the machine, not the code (thrash, disk,
                     runaway) → system-performance-remediation (if installed;
                     otherwise standard host triage) is the lever.
                     Exit: users are okay, the pager is quiet, and every
                     mitigation applied is written down with its own rollback.
Phase 1  EVIDENCE    trace-it on the preserved state and (if needed) live traps:
                     minimal probes until the trace SHOWS the bad state. Its gate
                     is this phase's exit: a capture in hand, not a theory. Hangs
                     and interleavings route through deadlock-finder-and-fixer /
                     gdb-for-debugging (if installed; otherwise a debugger by
                     hand) as trace-it's probe classes.
Phase 2  ROOT CAUSE  One sentence naming the broken invariant, no-slop grade:
                     "the cache key omits tenant id, so tenant A reads tenant B's
                     response." "Error handling was insufficient" does not exit
                     this phase. The sentence + the capture are the phase output.
Phase 3  FIX         Write the regression test FIRST and watch it FAIL on the
                     incident code (check out the pre-revert SHA / flip the flag
                     on in a sandbox; Phase 0's mitigation usually removed the
                     bug from HEAD, and the test must meet the actual bug). Then
                     fix THE bug via no-slop-bug-hunter (minimal diff, no
                     drive-bys) under the green ratchet. Exit: the test flips
                     red→green on the fix, suite green.
Phase 4  PREVENT     Mandatory, before close: route the CLASS to a durable guard,
                     at least one, chosen by shape (references/STABILIZE.md Part 4):
                     bug-zero scoped to the class ("where else does this invariant
                     break?") · test-max on the subsystem whose suite let it
                     through · a monitor/alert that would have paged BEFORE users
                     noticed (often a trace-it probe promotion) · skillify when
                     the method generalizes across projects · migrate-proof when
                     the real cause was an ungated migration. Exit: the prevention
                     artifact exists, is named, and is verified to trigger (an
                     alert you never test-fired is theater).
Phase 5  CLOSE       Ship the fix through ship-it (prod deploy is its button, not
                     yours). Post-deploy smoke on the real flow. Unwind Phase 0
                     mitigations deliberately (un-revert, remove the flag); a
                     forgotten emergency flag is next quarter's incident. Write
                     the incident record. Close ONLY when all four exist:
                     timeline · root-cause sentence · red→green regression test ·
                     prevention artifact.
```

## Stopping condition

Closed = the exit bar of Phase 5, all four artifacts. Not "the graphs look better,"
not "we know what happened," not a post-mortem doc with action items nobody owns.
An incident that reaches Phase 3 but stalls at Phase 4 stays OPEN in the decision
inbox with its prevention options listed; the conductor never quietly downgrades
prevention to a TODO.

## Skills it orchestrates

100% your arsenal: this skill is sequencing plus two gates. **Stabilize** with
git/flags/ship-it rollback paths + system-performance-remediation if installed
(+ memguard when the fire is memory on a shared, memory-constrained box);
**evidence** with trace-it (whose probe classes pull in gdb-for-debugging and
deadlock-finder-and-fixer if installed); **root-cause + fix** with
no-slop-bug-hunter; **prevent** with bug-zero / test-max / skillify /
migrate-proof; **ship + smoke** with ship-it. Full stage map and the mitigation
ladder in [references/STABILIZE.md](references/STABILIZE.md).

## Shared mechanisms (inherited from project-autopilot)

Green ratchet, decision inbox, capability-aware execution: as in project-autopilot
(see its CONVERGENCE / SAFETY-RAILS / EXECUTION). **Integration:** forge's DETECT
step routes a burning project here before any stage logic; when the fire is out and
the incident closed, forge resumes normal stage detection.

## THE EXACT PROMPT

```
/incident-zero
Target:   <project path>
Symptom:  <what's broken, as observed — alerts, user reports, error rates>
Severity: <users-down | degraded | data-at-risk>
Access:   <what I can touch: deploy history, flags, logs, prod shell, dashboards>
```

## Anti-Patterns

- Debugging before stabilizing: the cardinal sin. Every minute of root-causing
  during the fire is billed to users.
- Restarting/reverting before capturing what the restart destroys.
- Pulling an irreversible lever (data surgery, failover, destructive restart)
  without the operator's per-action go because "it's an emergency." Emergencies
  are why the button rule exists.
- A regression test written after the fix and never seen red on the incident code.
  It pins the fix's behavior, not the incident.
- Fixing the symptom the mitigation already hid and calling it the root cause.
- Closing on "fixed" with no prevention artifact: the class ships again.
- A prevention alert that was never test-fired.
- Leaving Phase 0's emergency flags/reverts in place indefinitely. Mitigations
  are debt with a pager attached; unwind them deliberately in Phase 5.
- Turning the incident record into prose theater. Four artifacts, verifiable,
  or it's open.

## References

- [STABILIZE.md](references/STABILIZE.md): the mitigation ladder, evidence-
  preservation checklist, the fail-first protocol, and the prevention router
