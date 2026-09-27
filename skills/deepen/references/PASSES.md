# Pass templates, the saturation test, and the PASSLOG format

## Pass 1: broad (the skill as designed)

```
Apply $<SKILL> to <TARGET>.
Scope: <TARGET> only — do not widen.
Output your findings as a numbered list, each tagged material / minor / nit,
each with file:line (or screen/flow) so a fix can land without re-discovery.
```

## Pass k ≥ 2: the deepening charge (PASSLOG prepended)

```
This is pass <k> of an iterative deepening run of $<SKILL> over <TARGET>.
Below is the log of every prior pass: what was found, what was fixed, what was
reverted and why.

<PASSLOG.md contents>

Everything in that log is SETTLED. Do not re-report it, do not re-litigate
reverted items, do not rephrase a settled finding as a new one. Your job is to
go DEEPER than every prior pass:
- categories and dimensions no prior pass covered
- edge cases, error paths, and states beneath the surface-level fixes
- problems the prior fixes themselves introduced or exposed
- the next layer down: what only becomes visible now that the obvious is fixed

If, after genuinely looking, there is nothing materially new — SAY SO
explicitly ("no material findings this pass") rather than manufacturing nits
to fill a list. An honest empty pass is the loop's success condition.

Output: numbered findings, material / minor / nit, file:line each.
```

## The saturation test (run after every pass's fixes are banked)

Saturation is diminishing returns per token, not an empty list. The pass is
saturated when ALL of:

1. Zero **material** findings (minor/nit-only passes count as saturated).
2. Nothing new in *kind*: every finding is a variation of a settled theme.
3. Effort-to-yield has clearly inverted vs. the prior pass (the pass worked
   hard to find little).

Two judgment rules:
- One stray material finding in an otherwise-dry pass: fix it, then run **one**
  more pass to confirm dryness. A single ember is not a fire, but check.
- A pass that surfaces a genuinely new *direction* (not a finding but a different
  class of work) is not saturation: finish this loop honestly, then report the
  direction to the caller rather than chasing it in-loop.

## PASSLOG.md entry format (append one block per pass)

```
## Pass <k> — <skill> — <timestamp>
Findings: <n> (<m> material / <x> minor / <y> nit)
1. [material] <finding> → FIXED <commit-ish>
2. [material] <finding> → REVERTED (broke <test>; 2 repair attempts failed)
3. [minor]    <finding> → FIXED <commit-ish>
4. [nit]      <finding> → DEFERRED (<one-line reason>)
Green: yes (build + tests + smoke) · Checkpoint: <commit-ish>
Saturation: no — material findings still landing
```

Final line of the file on exit:

```
EXIT: saturated | cap-hit | stop-rule — after pass <k> of cap <N>
```

Every finding gets exactly one terminal state: FIXED, REVERTED, or
DEFERRED-with-reason. Silence about a dropped finding is forbidden; fixing
every nit is not required.
