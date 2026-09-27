---
name: design-intel
description: >-
  Design intelligence for NEW UI surfaces: pick a named visual direction (style,
  palette, type pairing, layout system, component patterns, charts) and emit an
  actionable token-level design spec a builder can implement. Taste as a system,
  not generic defaults. Use when "design this dashboard/page/screen", "what should
  this look like", "pick a palette", "choose the fonts", "design direction",
  "make it feel like Stripe/Linear", building a new surface from scratch, or when
  ux-maximizer / project-autopilot route design-intelligence passes here.
---

# Design Intel

The decision layer above polish. ux-scan finds what's broken, polish-pass sands
what exists; this skill decides **what the thing should look like and why**,
before (or beneath) either. Given a surface to design, it commits to one named
visual direction and cashes it out as concrete tokens: palette, type pairing,
spacing scale, layout system, component choices, chart treatment. The output is
a spec a builder (human or agent) can implement without making a single taste
call of their own.

> **Core Insight:** Generic defaults are how AI-built UIs all look the same:
> blue-500, Inter, a card grid, done. Taste starts with a **commitment**: name
> one direction and let *every* token derive from it, so the surface is coherent
> instead of assembled. The direction is **sticky**. Pass 1 chooses it, every
> later pass refines *within* it (deeper hierarchy, better contrast, tighter
> states), never flips it. And premium is mostly subtraction: one accent color,
> two typefaces, neutrals doing 90% of the work, restraint at Stripe/Linear
> level. A bold direction executed with discipline beats a tasteful mood board
> every time.

## Preconditions / guards

1. **A surface to design.** A new page/screen/app/component family. This skill
   designs forward. Cleaning up an existing mess is ux-scan + polish-pass work.
2. **Stack identified.** Default target is Next.js + React + Tailwind +
   shadcn/ui (emit Tailwind tokens + shadcn component picks); Expo/RN gets
   StyleSheet/NativeWind equivalents. Read the repo before assuming.
3. **Existing design language wins.** If the project already has a committed
   direction (tokens, a prior `.design/DIRECTION.md`, an established brand),
   new surfaces extend it. Proposing a fresh direction for one page of a
   styled app is a coherence bug, not creativity.
4. **Greenfield direction = taste call.** A brand-new product's direction is
   the user's to approve: present the pick + runner-up (decision inbox or direct),
   don't auto-bank it. Extending an established language is auto.

## The Pipeline (staged: issue each phase only when the prior finishes)

### Phase 0: Ground
Read the project: what the surface is for, who uses it, in what mood (a
monitoring dashboard at 2am is not a marketing page). Inventory the existing
design language: tokens, tailwind config, globals.css, installed shadcn
components, any prior `.design/` artifacts. Detect the stack. Note what data
the surface shows (tables? time series? money?); that decides the chart layer.

### Phase 1: Direction
Choose a **named direction** from the starter library (references/LIBRARY.md)
or compose one. Never "clean and modern." Shortlist 2–3 candidates that fit
the surface's job and audience, reason each through (what it signals, what it
costs, how it ages), commit to one with rationale. Write
`<project>/.design/DIRECTION.md`, the sticky contract every later pass and
every sibling surface inherits. Greenfield → this is the user's approval gate (guard 4).

### Phase 2: System (tokens)
Derive everything from the direction, concretely:
- **Palette:** semantic CSS variables in shadcn convention (`--background`,
  `--foreground`, `--primary`, `--muted`, `--destructive`, ...), light + dark,
  actual OKLCH/hex values. One accent. Verify every fg/bg pair ≥ 4.5:1 (3:1
  for large text). A palette that fails contrast is not a palette.
- **Typography:** a named pairing (display + body, or one family two optical
  roles), a modular scale (give the actual px/rem steps), weights, line
  heights, where tabular-nums applies.
- **Layout:** grid system (12-col, bento, sidebar+canvas), container widths,
  a spacing scale (stick to Tailwind's; name the rhythm: section/card/element
  gaps), radius scale, shadow/elevation policy, border-vs-shadow stance.
- **Motion:** durations, easing, what animates and what never does.

### Phase 3: Components + charts
Map the surface's needs to **specific shadcn/ui components** with variant and
composition notes (which Card anatomy, which Table density, Dialog vs Sheet vs
Drawer and why). If the shadcn MCP is available, search it for the components
and pull real examples instead of guessing APIs. Where data viz exists: pick
chart types per question-the-user-is-asking (LIBRARY.md has the mapping),
default Recharts via shadcn charts, derive series colors from the palette,
strip chartjunk (no 3D, no rainbow, gridlines whisper).

### Phase 4: The spec
Write `<project>/.design/specs/<surface>.md`: direction recap → token blocks
ready to paste (CSS variables / tailwind config) → per-region layout (annotated
ASCII or structured description) → component manifest with states (hover,
focus-visible, loading, empty, error: specified, not implied) → chart specs →
an implementation order. The test: a builder with zero taste and this spec
ships something coherent.

### Phase 5: Build (Mode: build only)
When routed here to design *and* build (project-autopilot's "new UI surface"
route), implement the spec under the green ratchet: tokens first, then layout
shell, then components, checkpoint per slice. Then **look at it**: run the
app, screenshot the surface, check it against the spec's intent. Spec mode
stops at Phase 4 and hands off.

## Re-run semantics (how deepen passes work)

This skill is re-runnable with progressive deepening; ux-maximizer's cascade
runs it 3× via `/deepen`. The discipline:

- **Pass 1:** full pipeline: choose direction, build system, write spec.
- **Pass k ≥ 2:** read `.design/DIRECTION.md` + existing specs first. The
  direction is settled; go *deeper within it*: hierarchy sharpened, contrast
  pairs re-verified, state coverage completed, type scale tightened, spacing
  drift reconciled, chart treatments unified, dark mode parity, the places
  where the implementation quietly diverged from the spec.
- A pass may conclude the direction itself is wrong. That's a flag to the
  caller (or the user), never a silent re-roll. Saturation = a pass that only
  rephrases settled tokens.

## Contract (what orchestrators can rely on)

- **ux-maximizer / deepen:** 3× design-intelligence passes (styles, palettes,
  type, layout) land here. Each pass is coherent and self-contained, refines
  rather than restarts (direction sticky), and its deltas are loggable to
  PASSLOG.md.
- **project-autopilot:** routes "New UI surface to design/build" here, auto,
  except greenfield direction choice, which goes to
  `<project>/.autopilot/DECISIONS.md` (recommended option first).
- **Artifacts:** `<project>/.design/DIRECTION.md` (one per product, sticky) ·
  `<project>/.design/specs/<surface>.md` (one per surface). Both survive the
  run: they're the product's design memory, not scratch.
- **Pairs with:** ux-scan (finds problems) · polish-pass (elevates execution) ·
  premium-palette-design, if installed (deep palette work when the palette IS
  the task; otherwise do it in Phase 2).
  design-intel owns the *decide* layer; those own find and finish.

## THE EXACT PROMPT

```
/design-intel
Target:    <project path>
Surface:   <what to design, e.g. "admin analytics dashboard" | "marketing landing">
Mode:      spec | build   (default: spec)
Direction: <optional: force/keep a named direction; default: choose (sticky after pass 1)>
```

## Anti-Patterns

- "Clean and modern": that's the absence of a direction. Name one or compose
  one; every token must trace back to it.
- Generic defaults: blue-500 primary, Inter-for-everything, uniform card grid,
  shadow-md on all the things. If the spec could describe any app, it describes
  no app.
- Re-rolling the direction on pass 2 because a different one seems fresher.
  Direction is sticky; flag it, don't flip it.
- A new direction for one page of an already-styled product. Extend the
  existing language (guard 3).
- Auto-banking a greenfield direction without the user's gut-check. Taste stays
  the user's.
- Mood-board specs: "elegant, airy, premium" with no hex values, no scale
  steps, no component names. If a builder still has to make taste calls, the
  spec isn't done.
- Shipping a palette that fails contrast because it "looks better". 4.5:1 is
  the floor; work the lightness until it passes.
- Three accents, four typefaces, five radii. Premium is subtraction.
- Decorating charts (3D, rainbow series, heavy gridlines) instead of choosing
  the chart that answers the user's actual question.
- Specifying only the happy path. Hover, focus-visible, loading, empty, and
  error states are part of the design, not the implementer's problem.
- Guessing shadcn component APIs when the shadcn MCP is sitting right there.

## References

- [LIBRARY.md](references/LIBRARY.md): the starter library: named directions
  with palettes + type pairings + layout systems, the chart-selection map, and
  token emission templates (Tailwind/shadcn + Expo/RN)
