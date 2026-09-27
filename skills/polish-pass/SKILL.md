---
name: polish-pass
description: >-
  Iterative visual elevation of a UI that already works and looks decent. Per
  pass, tighten the details that separate "fine" from "premium": spacing
  rhythm, type hierarchy, contrast and interaction states, elevation, radii,
  motion, empty/loading/error states, density. Elevates the EXISTING design
  language toward Stripe/Linear-level craft; never a redesign. Use when
  "polish pass", "visual elevation", "tighten the UI", "make it feel premium",
  "it works but looks fine, not great", "Stripe-level", or when ux-maximizer's
  cascade or project-autopilot's "works but needs visual elevation" route lands here.
---

# Polish Pass

Take a screen that's *fine* and make it *premium* without changing what it is.
The gap between a decent UI and a Stripe/Linear one isn't a redesign; it's a few
hundred small correct decisions: every gap on the spacing scale, one type ramp
obeyed everywhere, a focus ring on everything focusable, shadows that agree about
where the light is, an empty state that was actually designed. This skill makes
those decisions, one screen at a time, one layer per pass.

> **Core Insight:** Polish is *convergence toward the design language the app
> already has*. Extract its real scales (spacing, type, radii, shadows, palette)
> first, then snap every deviation to them; never invent a parallel system. And
> repetition only compounds if each pass works a **deeper layer**: structure,
> then surface, then states, then motion, then micro. A pass that re-tightens
> last pass's padding is churn; a pass that finds the missing disabled state
> beneath last pass's now-clean layout is elevation. Restraint over flash:
> the premium move is almost always *less*: fewer borders, fewer font sizes,
> shorter durations, quieter color.

## Preconditions / guards

1. **The app works and looks decent.** Polish is a maturity activity. Broken
   function → fix first; no coherent design language at all → that's
   `design-intel` (direction) territory, not polish.
2. **Green baseline + checkpoint.** Build + tests + smoke green, checkpoint
   committed, before the first edit. Know the verify command. A polish change
   that breaks it gets reverted, however good it looked.
3. **Scope is named screens.** "Polish the app" is not a scope. Get (or pick and
   state) the key screens before starting; this skill operates screen-by-screen.

## The deepening ladder (what each pass works)

Pass k climbs to rung k. Settled rungs stay settled; re-litigating them is churn.

```
1  STRUCTURE  spacing rhythm on ONE scale · alignment to a real grid · type
              ramp (≤ 5 sizes/screen) · hierarchy: one primary action per view
2  SURFACE    color/contrast (AA minimum) · elevation: shadows form a coherent
              ramp, one light source · border-radii: one nesting-consistent set
3  STATES     hover / focus-visible / active / disabled on every interactive
              element · empty, loading (skeleton > spinner), and error states
              exist and are DESIGNED, not defaulted
4  MOTION     transitions on state changes (150–250ms, ease-out for enter) ·
              nothing animates layout that could animate transform/opacity ·
              prefers-reduced-motion respected · no springy flash
5  MICRO      optical alignment (icons, baselines) · density tuning · tabular
              numbers · truncation behavior · 1px-level nudges · copy de-slop
```

Per-rung checklists with Tailwind/shadcn-specific moves: references/CRAFT.md.

## The Pipeline (staged: per pass, per screen)

### Phase 0: Extract the language
Read the project's actual design system before touching anything: `tailwind.config.*`
/ `globals.css` tokens, the shadcn theme, the most-polished existing screen (it's
the reference implementation of intent). Write down the real scales: spacing
steps, type ramp, radii set, shadow set, palette roles. Every fix in later phases
snaps to *these*, never to a generic ideal. Confirm green, checkpoint.

### Phase 1: Sweep one screen
Walk this pass's rung over the screen with the running app open, not just the code.
Number the findings with impact (material / minor / nit). A finding names the
element, the deviation, and the language-conformant fix.

### Phase 2: Fix under the green ratchet
Smallest diff that snaps the element to the language. One finding at a time;
verify between fixes; a fix that breaks build/tests/smoke gets 2 repair attempts
then reverts. Mid-fix temptations to go bolder ("this whole sidebar should be...")
are **direction calls**: queue them to the decision inbox / `design-intel`, don't
freelance them. Repeat Phase 1–2 for each named screen before the pass ends.

### Phase 3: Look at it
Run the app; capture before/after of each screen (use /run or /verify if
available; otherwise take the screenshots by hand). Tests prove it still works;
only eyes prove it got better. Anything that reads worse in the screenshot than
it did in the diff: revert it. The screenshot wins.

### Phase Ω: Bank
Final verify green, checkpoint, short report: rung worked, findings fixed /
reverted / deferred per screen. Under `/deepen`, this lands in PASSLOG.md.

## Contract (what orchestrators can rely on)

- **`/deepen` wraps this skill** for counted passes. ux-maximizer's cascade
  step "5× UI polish on the key screens" is `/deepen Skill: polish-pass
  Passes: 5`, and the 5-rung ladder is what makes those five passes deepen
  instead of churn: deepen's PASSLOG carries the settled rungs forward; this
  skill defines what "deeper" means for visual craft.
- **project-autopilot** routes "works but needs visual elevation" here at
  auto tier. Routine craft fixes never need the operator; direction changes always do.
- **Siblings:** `ux-scan` finds the problems, `design-intel` sets the
  direction, and polish-pass is the craft-elevation layer between. It executes
  toward the language the other two establish.
- **Green exit, always.** Hands back a working app or reverts to checkpoint.
- **Screen-by-screen, never global.** No app-wide find-replace "polish";
  each screen gets a focused sweep and its own before/after.

## THE EXACT PROMPT

```
/polish-pass
Target:  <project path>
Screens: <key screens/routes, e.g. "/dashboard, /settings" — or "auto" to pick>
Rung:    <1–5 ladder rung — or "auto": next unworked rung per PASSLOG>
```

## Anti-Patterns

- Redesigning under the flag of polish. New palette, new layout, new component
  library is `design-intel` work and an ask-user decision, full stop.
- Inventing a one-off value (`p-[13px]`, a fifth shadow, a new gray) instead of
  snapping to the nearest step of the extracted scale.
- Adding flash (gradients, glows, bouncy springs) where the premium move is
  restraint. Stripe is quiet; quiet is the bar.
- Polishing the happy path and shipping default empty/loading/error/disabled
  states. Rung 3 exists because that's where "fine" apps stay fine.
- Pass 3 re-tightening pass 1's spacing. Settled rungs are settled; go deeper.
- App-wide regex polish across all screens at once. Screen-by-screen or it's
  not a sweep, it's a spray.
- Keeping a polish change that broke a test because it looked right.
  Right-but-red is red; revert.
- Declaring a pass done from the diff without looking at the running screen.

## References

- [CRAFT.md](references/CRAFT.md): per-rung checklists with the concrete
  Tailwind / shadcn / React moves, and the saturation test for visual passes
