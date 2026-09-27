---
name: ux-scan
description: >-
  Systematic UX evaluation of a UI: Nielsen's 10 usability heuristics +
  accessibility (contrast, keyboard nav, focus order, ARIA, screen-reader
  labels, touch targets) + interaction/flow problems, emitted as a ranked list
  of located, fix-ready findings. Use when "review the UI", "is this usable",
  "usability check", "accessibility audit", "a11y pass", "find the UX
  problems", "heuristic evaluation", pre-launch UX review, or when
  ux-maximizer / deepen / project-autopilot need their UX-evaluation stage.
---

# UX Scan

Walk a UI the way a usability expert and a screen-reader user both would, and
come back with a numbered, ranked list of concrete problems, each one located
(file:line + screen), tied to the heuristic or WCAG criterion it violates,
severity-rated, and paired with the specific fix. The scan *surfaces*; a fixer
(the caller, a deepen pass, or a Fix: yes tail) *resolves*. That separation is
the whole design.

> **Core Insight:** A UX finding is worth exactly as much as its address. "Improve
> the contrast" is advice; "`text-gray-400` on `bg-white` in
> `components/PricingCard.tsx:34` is 2.5:1, needs 4.5:1; use `text-gray-600`"
> is work a fixer can bank in one move. So every finding ships the full tuple
> (WHAT / WHERE / WHY (criterion) / SEVERITY / FIX), and the scan itself stays
> **read-only**: it never edits the artifact it's inspecting mid-scan, which is
> what makes it cleanly re-runnable. Pass N reads what passes 1..N−1 settled and
> spends its whole budget on new territory.

## Preconditions

1. **A reachable UI.** The scan must *see* the app, not just the code. Run it
   (/run or /verify if installed, otherwise start the app yourself; screenshot
   key flows for webapps). Code-only scanning
   misses everything about how the thing feels; eyes-only scanning misses
   everything about why. Do both.
2. **Stack identified.** The pack's target UIs skew Next.js + React + Tailwind (+
   shadcn/ui), some Electron, some Expo/React Native. The checks in
   references/CHECKLIST.md are tuned per stack; know which you're on before
   Phase 1.
3. **Prior findings read.** If `<target>/.uxscan/FINDINGS.md` exists (or a
   deepen PASSLOG charge is attached), settled findings are off the table;
   re-reporting them wastes the pass.

## The Pipeline (staged: issue each phase only when the prior finishes)

### Phase 0: Orient
Identify the stack and enumerate the surface: routes/screens, the 2–4 flows a
real user actually runs (onboard, core action, settings, destructive action),
and the component inventory. Run the app and walk those flows once with eyes
before reading a line of source. Note the user the app serves: severity is
judged against *their* task, not abstract correctness.

### Phase 1: Heuristic sweep (Nielsen's 10)
Walk each key flow against all ten heuristics (expanded with stack-specific
tells in references/CHECKLIST.md): visibility of system status, match to the
real world, user control/freedom, consistency, error prevention, recognition
over recall, flexibility/efficiency, minimalist design, error recovery, help.
Log every violation as a draft finding with its location.

### Phase 2: Accessibility sweep (WCAG)
Code-level + behavioral, in this order: **contrast** (measure ratios, never
eyeball: 4.5:1 text, 3:1 large/UI), **keyboard** (tab through every flow,
mouse unplugged: reachable, operable, no traps), **focus order + visibility**
(logical sequence, visible ring; Tailwind's `focus:outline-none` without a
replacement is an automatic finding), **ARIA + names** (icon-only buttons need
labels; shadcn/Radix gives you semantics free, composed raw `div onClick`
throws them away), **screen-reader labels** (images `alt`, inputs associated
labels, RN `accessibilityLabel`/`accessibilityRole`), **touch targets** (44pt
iOS / 48dp Android / 24px WCAG floor, plus RN `hitSlop`). Run the automated
floor too (axe via Playwright, eslint-plugin-jsx-a11y). Tools catch ~30–40%;
they are the floor of this phase, never the phase.

### Phase 3: Interaction & flow sweep
The problems heuristic lists under-weight: missing loading/empty/error states
(Next.js routes without `loading.tsx`/`error.tsx`), forms that lose input on
failure or validate only on submit, destructive actions without confirm/undo,
dead-end states with no exit, layout shift and jank, mobile breakpoints that
trap content, latency with no feedback past ~400ms, and copy that names the
system's internals instead of the user's task.

### Phase 4: Verify, rank, emit
Re-check every draft finding against the running app. A finding you can't
reproduce gets cut, not hedged. Dedupe (one root cause = one finding, listing
all its sites). Severity-rate each: **blocker** (users fail or are excluded) /
**major** (users struggle or workaround) / **minor** (friction) / **nit**
(polish), rated by user impact × frequency, not by how interesting the bug
is. Write `<target>/.uxscan/FINDINGS.md` (format below), ranked
blocker-first. The scan ends here.

### Phase 5: Fix tail (only if `Fix: yes`)
Standalone invocations may fix: green baseline + checkpoint first, then
resolve findings top-down one at a time under the green ratchet (a fix that
breaks build/tests/smoke gets repaired or reverted; right-but-red is red),
re-verifying each fix *by looking*, not just by passing. Under deepen, skip
this phase; deepen's own loop body owns fixing.

## The finding (Phase 4 template)

Every finding self-contained: a fixer with zero context can execute it:

```
UX-<n>  [severity]  <one-line what>
WHERE   file:line + route/screen + how to reproduce
WHY     heuristic (N1–N10) or WCAG SC (e.g. 1.4.3) + the user consequence
FIX     the specific change: component, class, prop, or pattern to apply
```

## Contract (what orchestrators can rely on)

ux-maximizer runs this 3× over the whole app via **deepen** (each pass's
findings fixed before the next fires), and project-autopilot routes
"Usability/heuristic evaluation" here on auto. The interface is fixed:

- **`<target>/.uxscan/FINDINGS.md`:** ranked findings, stable `UX-<n>` IDs,
  appended per scan under a `## Scan <k> — <date>` header; fixed items get
  `[fixed]` on their line so later scans treat them as settled.
- **Read-only by default:** the scan never mutates the target unless
  `Fix: yes`. Deepen's loop can therefore trust the artifact it audits.
- **Re-runnable:** each scan reads prior sections, carries still-open IDs
  forward in one line, and spends its effort on territory the prior passes'
  fixes opened up or didn't cover.
- **Severity maps to deepen's impact scale:** blocker/major → material,
  minor → minor, nit → nit.

## THE EXACT PROMPT

```
/ux-scan
Target: <project path or running app URL>
Scope:  <optional: one flow/screen, e.g. "checkout"; default whole app>
Fix:    <no (default: surface only) | yes (fix top-down under green ratchet)>
```

## Anti-Patterns

- Findings without an address. "Improve feedback" is advice, not a finding.
- Fixing mid-scan: you mutate the artifact under inspection and corrupt the
  rest of the pass; surface everything, then fix (or hand off).
- Re-reporting settled findings on a re-run. Pass N's budget belongs to new
  territory, not re-litigating pass N−1.
- Eyeballing contrast. Measure the ratio; `gray-400`-on-white *looks* fine on
  a calibrated display and fails for half of real users.
- Auditing with a mouse in hand. Phase 2's keyboard walk is done keyboard-only
  or it isn't done.
- Treating axe/lint output as the audit. Tools are the floor; heuristics,
  flows, and judgment are the audit.
- Severity inflation: a wall of blockers ranks nothing; rate by user impact ×
  frequency and let most findings be minor, because most are.
- Code-only scanning: if you never ran the app, you scanned a hypothesis.
- Padding a clean pass. Few findings on a re-run is the system working
  (saturation), not under-delivery; never manufacture nits to look thorough.

## References

- [CHECKLIST.md](references/CHECKLIST.md): Nielsen's 10 with stack-specific
  tells, the WCAG criteria list, the interaction/flow problem catalog, and
  per-stack (Next.js/Tailwind/shadcn, Electron, Expo/RN) check patterns
