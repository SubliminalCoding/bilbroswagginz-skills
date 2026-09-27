# Craft Checklists: Polish Pass

Per-rung checklists with concrete moves for the pack's primary stack (Next.js +
React + Tailwind + shadcn/ui; same principles port to Electron/mobile). Every
fix snaps to the scales extracted in Phase 0. These lists tell you *what to
check*; the project's own language tells you *what value to use*.

## Rung 1: STRUCTURE

**Spacing rhythm**
- Every margin/padding/gap is a step on ONE scale (Tailwind's 4px scale unless
  the project overrides). Arbitrary values (`p-[13px]`, `mt-[7px]`) → nearest step.
- Related elements closer than unrelated ones: proximity IS grouping. If a
  label sits equidistant from its field and the next field, the gap above grows.
- Section gaps > card gaps > intra-card gaps; the ramp is monotonic.
- Symmetry where the eye expects it: card padding equal on all sides unless
  there's a designed reason (e.g. tighter under a header).

**Alignment**
- One left edge per column of content: text, icons, inputs share it. Hunt the
  stragglers with the browser's element outlines, not by eyeball alone.
- Baselines align across columns (avatar text next to its avatar, stat labels
  across stat cards). `items-baseline` over `items-center` for mixed type sizes.
- Page gutters consistent across screens (`max-w-*` + `mx-auto` + same `px-*`).

**Type ramp & hierarchy**
- ≤ 5 font sizes per screen. Off-ramp sizes (`text-[15px]`) → nearest ramp step.
- Weight + size + color do hierarchy together: one obvious primary heading,
  secondary text actually secondary (`text-muted-foreground`, not just smaller).
- Line-height: tight for headings (`leading-tight`), relaxed for body
  (`leading-relaxed` for paragraphs); line length ≤ ~75ch for reading text.
- One primary action per view: exactly one filled/default Button variant in a
  given region; everything else `secondary` / `outline` / `ghost`.

## Rung 2: SURFACE

**Color & contrast**
- Body text ≥ 4.5:1 against its background; large text and UI glyphs ≥ 3:1.
  Check the muted grays; that's where AA quietly fails.
- Color roles consistent: one semantic per hue (destructive = red everywhere,
  not red-for-delete here and red-for-badge there).
- Grays from one family. Mixed `slate`/`gray`/`zinc` on a screen reads dirty.
  Pick the project's family (shadcn theme base) and converge.
- Borders quiet: prefer `border-border` at low contrast or a shadow over a hard
  line; never both a strong border AND a strong shadow on the same surface.

**Elevation**
- Shadows form a coherent ramp (`shadow-sm` → `shadow` → `shadow-md` ...), each
  level meaning one thing (resting card / raised card / popover / modal).
- One light source: all shadows agree on direction. No ambient mega-blur on one
  card and crisp drop on its neighbor.
- Dark mode: elevation reads through *surface lightness* (lighter = higher),
  not shadow strength. Verify both themes.

**Border-radii**
- One radius set, derived from the theme token (`--radius` in shadcn). No screen
  mixing `rounded-md` cards with one `rounded-2xl` outlier without intent.
- Nested radii: inner radius ≈ outer radius − padding (inner element rounder than
  its container looks broken).

## Rung 3: STATES

**Interactive states: every interactive element has all of them**
- `hover:`: perceptible but quiet (background shift, not a jump).
- `focus-visible:`: a visible ring (`focus-visible:ring-*`), never
  `outline-none` without a replacement. Tab through the whole screen to verify.
- `active:`: pressed feedback (slight darken / scale-[0.98] at most).
- `disabled:`: `disabled:opacity-50 disabled:pointer-events-none` (shadcn
  default) AND a reason available (tooltip/help text) when non-obvious.

**Async & edge states: designed, not defaulted**
- Empty: an icon/illustration, one line of explanation, and the action that
  fills it. Never a bare "No data".
- Loading: skeletons matching final layout (`Skeleton` sized to content) beat
  spinners; spinners only for sub-second or indeterminate actions. Buttons get
  an in-button spinner + disabled during submit. No layout shift on resolve.
- Error: human message + retry affordance, inline near the failure, not only
  a toast that evaporated.
- Overflow: long names, 0 items, 1 item, 10k items. Truncate with `truncate` +
  `title`/tooltip, and decide wrapping deliberately.

## Rung 4: MOTION

- State changes transition; appearing/disappearing content fades/slides 150–250ms.
  Enter `ease-out`, exit `ease-in`, hovers ~150ms. > 300ms feels sluggish; premium
  motion is barely noticed.
- `transition-colors` / `transition-opacity` / `transition-transform`: scope the
  transition; bare `transition-all` invites jank.
- Animate `transform`/`opacity`, not layout properties (no animating `height`
  where a transform works; for height, prefer grid-rows or Radix/shadcn
  collapsible primitives that handle it).
- Respect `prefers-reduced-motion` (`motion-reduce:transition-none` or the
  global media query).
- No decorative loops, no spring overshoot on routine UI. Motion explains
  (where did this come from / go); it never performs.

## Rung 5: MICRO

- Optical alignment: icons optically centered next to text (often 1px nudge),
  chevrons aligned to the text baseline grid, icon sizes from one set (16/20/24).
- Numbers that compare or update: `tabular-nums`. Currency/dates formatted
  consistently (one formatter, one locale source).
- Density: gauge against intent. Data-dense screens (tables, dashboards) tighten
  toward Linear; marketing/empty-ish screens breathe toward Stripe. Same control
  heights within a row (`h-9` inputs next to `h-9` buttons).
- Cursor honesty: `cursor-pointer` on clickable, not on disabled.
- Selection/scrollbar styling where the platform default jars in dark mode.
- UI copy de-slop: sentence case, no exclamation marks, verbs on buttons
  ("Create project", not "OK"); kill placeholder copy that survived.

## The saturation test (when wrapped by /deepen)

A polish pass is saturated when its findings are all nits: values already on
the scale being shuffled to adjacent steps, motion durations bikeshedded ±25ms,
re-phrasings of settled items. Material findings name a *perceivable* defect a
user could feel even without naming it. No material findings at the current
rung and no rungs left → saturated; rungs left → next pass climbs rather
than grinds.

## What this skill never does

Change the brand palette, swap fonts, restructure navigation/IA, replace the
component library, or "modernize" the design language. Those are direction:
`design-intel` + the operator. Polish makes the existing language more *itself*.
