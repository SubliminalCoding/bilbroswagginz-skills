# Design Intel: Starter Library

Named directions, the chart-selection map, and token emission templates.
These are **starting points**: compose, tune, and derive from the surface's
job. The rule stands everywhere: one accent, two typefaces max, neutrals do
the heavy lifting, every fg/bg pair ≥ 4.5:1 (3:1 large text).

---

## Part 1: Named directions

Each entry: what it signals · palette skeleton · type pairing · layout ·
where it fits. Values are light-mode anchors; derive dark mode by flipping
the neutral ramp and *desaturating* the accent slightly (pure accents glow
ugly on dark).

### 1. Ledger
Quiet competence. The Stripe-docs feel: information-first, color is a tool.
- **Palette:** near-white paper `#FAFAF9`, ink `#1C1917`, one violet accent
  `#635BFF`-family, success/destructive only as semantic whispers.
- **Type:** Inter (UI) + a mono (JetBrains Mono / Geist Mono) for numbers,
  IDs, code. Tabular-nums everywhere money or counts appear.
- **Layout:** sidebar + canvas, 12-col content area, 1px borders over
  shadows, generous row height in tables.
- **Fits:** dashboards, billing, admin panels, anything with tables of money.

### 2. Terminal Luxe
Dark-native operator tool. Linear's discipline: dark isn't a theme, it's home.
- **Palette:** layered charcoals (`#08090A` → `#141516` → `#1D1E20` surfaces),
  text `#E6E6E6`/`#8A8F98`, a single cool accent (indigo `#5E6AD2` or cyan),
  thin `#FFFFFF0F` hairline borders.
- **Type:** one grotesk family (Inter / Geist) at two weights doing all roles;
  size restraint: 13–14px UI base, hierarchy via weight + color, not size.
- **Layout:** dense lists + command-palette mentality, keyboard affordances
  visible, 4px-grid spacing, radius 6–8px, elevation = lighter surface not
  bigger shadow.
- **Fits:** dev tools, agent consoles, monitoring, operator-dashboard surfaces.

### 3. Gallery
Editorial whitespace. The product is the art; chrome disappears.
- **Palette:** true white, near-black `#111`, NO accent chrome; color enters
  only through content/imagery; links underlined, not colored.
- **Type:** high-contrast serif display (Fraunces, Canela-class) + neutral
  sans body (Söhne-class / Inter). Big display sizes, tight tracking.
- **Layout:** asymmetric grid, oversized margins, few elements per viewport,
  scroll as pacing.
- **Fits:** portfolios, premium marketing, brand-led landing pages.

### 4. Bento Showcase
Apple-keynote energy: features as a wall of self-contained tiles.
- **Palette:** soft neutral field `#F5F5F7`, white cards, per-tile accent
  moments allowed (the one place multi-accent is legal: *inside* tiles,
  never in chrome).
- **Type:** SF-adjacent sans (Inter Display / General Sans), heavy display
  weights, short punchy copy.
- **Layout:** bento grid: mixed tile spans on a strict underlying grid
  (e.g. 6-col, tiles span 2/3/4/6), radius 16–24px, consistent gap (16/24px),
  every tile self-sufficient.
- **Fits:** feature pages, launch pages, "what it does" marketing sections.

### 5. Glass Panel
Depth via translucency. Use with discipline or it reads 2021.
- **Palette:** a quiet gradient or image field behind; panels
  `bg-white/10`–`/60` + `backdrop-blur-md/lg` + 1px `white/20` border; text
  must sit on the *panel*, never raw on the field.
- **Type:** clean geometric sans (Plus Jakarta, Outfit).
- **Layout:** floating panels over a field, large radius (16px+), one blur
  strength site-wide.
- **Fits:** consumer app marketing, music/media UIs, hero sections. Cap at
  2–3 glass layers; everything-glass is the anti-pattern.

### 6. Soft Clay
Friendly, rounded, tactile: approachable consumer warmth without childish.
- **Palette:** warm cream `#FAF7F2`, deep warm gray text, one saturated warm
  accent (coral/amber family), pastel tints for category color.
- **Type:** rounded-friendly sans (Nunito Sans, DM Sans) + same family bold
  display.
- **Layout:** large radius (16–24px), soft double shadows (tight dark + wide
  soft), pill buttons, chunky touch targets.
- **Fits:** consumer wellness/coaching apps, onboarding flows,
  anything talking to non-technical humans at a vulnerable moment.

### 7. Brutal Honest
Loud, flat, confident. Anti-slick on purpose.
- **Palette:** white/cream + black `#000`, 1–2 electric accents (yellow
  `#FFE600`, blue `#2E5CFF`), zero gradients.
- **Type:** heavy display (Archivo Black, Space Grotesk bold) + mono or plain
  sans body. ALL-CAPS labels legal here.
- **Layout:** visible 2px borders, hard offset shadows (`4px 4px 0 #000`),
  radius 0–4px, dense and unapologetic.
- **Fits:** dev-tool marketing, indie products, brands that want personality
  over polish. Never for finance/health trust surfaces.

### 8. Paper Terminal
Warm analog-meets-mono. Documentation and reading-first products.
- **Palette:** warm paper `#F7F3EC`, ink `#2A2722`, one burnt accent
  (rust/forest), code blocks on slightly deeper paper.
- **Type:** humanist serif body (Source Serif, Lora) + mono for everything
  technical; 16–18px body, 65–75ch measure.
- **Layout:** single readable column + sticky TOC, hairline rules, footnote
  energy.
- **Fits:** docs sites, long-form content, changelogs, books/PDF-derived UIs.

### 9. Signal Dark (data-viz first)
A dashboard direction where the charts are the interface.
- **Palette:** deep neutral `#0B0E14` field, muted chrome, a *categorical
  series ramp* designed first (5–6 distinguishable mid-saturation hues) and
  UI accent = series-1.
- **Type:** compact grotesk + mono for axis/values, tabular-nums mandatory.
- **Layout:** grid of chart cards with uniform header anatomy (title ·
  big-number · delta chip · sparkline/chart), one card = one question.
- **Fits:** analytics, ops monitoring, trading-adjacent, status pages.

### 10. Civic Clear
Maximum-legibility utility. Accessibility as the aesthetic.
- **Palette:** white, `#1A1A1A` text, one strong blue accent ≥ 7:1 on white,
  semantic colors textbook (green/amber/red) always paired with icon + text.
- **Type:** one workhorse sans (Public Sans, Inter) at a generous scale,
  16px floor.
- **Layout:** single-column forms, fat labels above inputs, obvious focus
  rings, no hover-only affordances.
- **Fits:** forms, government/health/insurance-adjacent, onboarding for
  broad audiences, anything where confusion has a real cost.

### 11. Neon Stage
Show-business dark: built to be *watched*, not used quietly.
- **Palette:** near-black stage, 1–2 neon accents (gold + cyan-class) with
  glow (text-shadow / outer glow) reserved for moments, not chrome.
- **Type:** condensed display (Bebas-class / Archivo Expanded) + clean sans.
- **Layout:** centerpiece-first (one dominant region), overlay/toast moments
  with blur+glow, motion is part of the design (entrances matter).
- **Fits:** livestream overlays, celebration moments, broadcaster UIs
  (on-stream toast alerts belong in this family).

### 12. Field Notes (mobile-first)
Calm native-feeling mobile, Expo/RN default.
- **Palette:** system-ish neutrals, one brand accent, semantic colors from
  platform conventions; dark mode is a first-class twin, not an afterthought.
- **Type:** platform stack (SF/Roboto via system font) + brand display used
  *sparingly* (headers only); never ship 3 custom font files to mobile.
- **Layout:** screen = scroll of full-bleed sections, 16px gutters, list
  rows over cards-everywhere, native nav patterns (tabs + stack), 44pt
  touch floor.
- **Fits:** Expo/RN apps, mobile companions to web products.

### Composing a direction
When none fits: pick a **base** (closest entry), name the **twist** (one
deliberate deviation, e.g. "Ledger but warm paper neutrals"), and write it
up in DIRECTION.md with the same skeleton (signals / palette / type / layout /
fits). One twist. Two twists is a new direction; three is a mood board.

---

## Part 2: Chart selection map

Choose by the question the user is asking, never by what looks impressive:

| User's question | Chart | Notes |
|---|---|---|
| How is X changing over time? | Line | ≤ 4 series; more → small multiples |
| How much, compared across categories? | Horizontal bar | Sorted by value, not alphabet |
| Time + categories together? | Grouped/stacked bar | Stack only when the total matters |
| Parts of a whole, right now? | Donut (≤ 5 slices) or stacked bar | > 5 slices → bar; never pie forests |
| Distribution / spread? | Histogram, box plot | |
| Correlation between two measures? | Scatter | Add trend line only if it's real |
| Single KPI + trend? | Big number + delta chip + sparkline | The dashboard workhorse |
| Flow between states? | Sankey | Rare; only when flow IS the story |
| Density over two dims / time grid? | Heatmap | Sequential ramp, not rainbow |
| Progress toward a target? | Bullet/progress bar | Skip gauges; they waste space |

**Treatment rules:** series colors derive from the direction's palette (build
the categorical ramp once, reuse everywhere); axis text = muted foreground at
11–12px tabular-nums; gridlines at ≤ 10% opacity, horizontal only when
possible; no 3D, no dual y-axes without a confession, zero-baseline for bars
always; empty/loading/error chart states specified like any component.
Default stack: Recharts via shadcn charts (web), Victory Native (Expo).

---

## Part 3: Token emission templates

### shadcn/ui CSS variables (globals.css): emit BOTH modes

```css
:root {
  --background: <oklch/hex>;
  --foreground: <...>;
  --card: <...>;            --card-foreground: <...>;
  --primary: <accent>;      --primary-foreground: <...>;
  --secondary: <...>;       --secondary-foreground: <...>;
  --muted: <...>;           --muted-foreground: <...>;
  --accent: <...>;          --accent-foreground: <...>;
  --destructive: <...>;     --destructive-foreground: <...>;
  --border: <...>; --input: <...>; --ring: <accent>;
  --radius: <direction radius, rem>;
  --chart-1: <series ramp...>; /* through --chart-5 */
}
.dark { /* full twin — every variable, accent slightly desaturated */ }
```

### Type + spacing spec block (paste into the surface spec)

```
TYPE     display: <family> <weights>   body: <family> <weights>   mono: <family>
SCALE    12 / 14 / 16 / 18 / 22 / 28 / 36 / 48  (trim to ~6 steps actually used)
LINE     body 1.5–1.6 · headings 1.1–1.2 · UI chrome 1.3
RHYTHM   page sections 64–96px · card padding 24px · element gap 8–12px
         (Tailwind steps only; no magic margins)
RADIUS   <one scale: e.g. 8 / 12 / 16> · SHADOW <policy: borders-first | soft-double | none>
MOTION   150–200ms ease-out micro · 250–350ms panels · respect prefers-reduced-motion
```

### Expo/RN equivalent
Emit the same semantic names as a `tokens.ts` theme object (light/dark),
spacing as a numeric scale (4-base), and platform font stacks. NativeWind
projects map the same CSS variables via its config.

### Component manifest line format (Phase 3 → spec)

```
<Region> · <shadcn component(s)> · variant/size · composition note
states: default / hover / focus-visible / loading / empty / error / disabled
```
