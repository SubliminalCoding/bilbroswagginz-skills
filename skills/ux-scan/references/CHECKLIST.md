# ux-scan: the working checklist

Three sweeps (heuristics, accessibility, interaction/flow) plus per-stack
check patterns. Every check ends in a located finding or a pass, never a
general remark.

---

## Sweep 1: Nielsen's 10, with stack-specific tells

**N1: Visibility of system status.** The user always knows what's happening.
Tells: async action with no pending state on the button; mutation with no
toast/optimistic update; Next.js route segment without `loading.tsx`; upload
or long job with no progress; stale data with no refresh indicator.

**N2: Match between system and the real world.** The UI speaks the user's
language. Tells: error copy that surfaces internals ("hydration failed",
"ECONNREFUSED", raw zod messages); developer nouns in nav labels; ISO
timestamps where "2 hours ago" serves; jargon the app's actual user wouldn't
say out loud.

**N3: User control and freedom.** Exits and undo everywhere. Tells: modal
with no Escape/overlay-click close (raw `div`, not shadcn `Dialog`);
multi-step flow with no back that preserves input; destructive action with no
undo *or* confirm; browser Back broken by client state (state not in the URL
where it should be).

**N4: Consistency and standards.** Same thing looks/acts the same everywhere.
Tells: mixed shadcn `<Button>` and hand-rolled button styles; primary action
left on one screen, right on another; two date formats; icon meaning X here
and Y there; external vs internal links indistinguishable.

**N5: Error prevention.** Stop the mistake before it happens. Tells: submit
enabled while required fields are empty; no inline validation before the
expensive step; free-text where a constrained input (select, date picker)
prevents the error class; double-submit possible (button not disabled
in-flight); destructive default focus in confirm dialogs.

**N6: Recognition rather than recall.** Options visible, not memorized.
Tells: bare icon buttons with no label or tooltip; keyboard shortcuts that
exist nowhere discoverable (Electron: missing menu accelerators); empty form
re-asking data the app already has; search with no recent/suggestions.

**N7: Flexibility and efficiency of use.** Accelerators for the experienced.
Tells: no keyboard path through the core flow; lists without bulk actions
where users repeat actions; no command palette in a power-user app; settings
that can't deep-link.

**N8: Aesthetic and minimalist design.** Every element earns its place.
Tells: competing primary CTAs; settings page dumping every option at one
level; decorative noise around the data; three font sizes doing one job;
density mismatched to the task (sparse dashboard, cramped reader).

**N9: Help users recognize, diagnose, recover from errors.** Tells: generic
"Something went wrong" with no cause or retry; Next.js segment without
`error.tsx` (or one without a `reset()` retry); form error at the top while
the bad field sits unmarked below the fold; error states that destroy the
user's input.

**N10: Help and documentation.** Tells: empty states that don't teach the
first action; complex fields with no example/hint; features whose only
documentation is trying them.

---

## Sweep 2: Accessibility (WCAG), in execution order

Run the automated floor first (axe-core via Playwright
(`npx playwright test` with `@axe-core/playwright`, or `axe` in DevTools) and
`eslint-plugin-jsx-a11y`), then do the manual majority it cannot catch.

**Contrast (SC 1.4.3 / 1.4.11).** Measure, never eyeball: 4.5:1 normal text,
3:1 large text (≥24px or ≥18.7px bold) and UI components/focus indicators.
Tailwind traps: `text-gray-400`/`text-gray-500` on white, `text-muted-foreground`
on tinted backgrounds, placeholder-as-label, disabled-looking text that's
actually enabled, dark-mode variants never re-checked. Check both themes.

**Keyboard (SC 2.1.1 / 2.1.2).** Tab through every key flow, mouse unplugged:
every interactive element reachable and operable (Enter/Space), no traps
(modals must contain focus *and* release it on close; free with shadcn
`Dialog`, manual with hand-rolled ones). React trap: `onClick` on `div`/`span`
means no focus, no key handling, no role. Electron: every menu action needs an
accelerator; no shortcut may shadow system ones.

**Focus order + visibility (SC 2.4.3 / 2.4.7).** Tab order follows visual/
logical order (DOM order vs CSS-order mismatch from `flex-row-reverse`/
`order-*`); focus visible on every interactive element; `focus:outline-none`
without `focus-visible:ring-*` is an automatic finding; after client-side nav
or modal close, focus lands somewhere sensible (trigger or new content), not
reset to `<body>`.

**Name, role, value (SC 4.1.2).** Every control has an accessible name:
icon-only buttons need `aria-label` or `sr-only` text; shadcn `Dialog` needs
`DialogTitle` (Radix warns; treat the warning as a finding); custom
comboboxes/menus composed from divs instead of Radix primitives re-implement
ARIA badly; the fix is usually "use the primitive". State exposed:
`aria-expanded`, `aria-selected`, `aria-current` for nav.

**Screen-reader labels (SC 1.1.1 / 1.3.1 / 3.3.2).** Meaningful images need
real `alt` (decorative: `alt=""`); inputs need associated `<Label htmlFor>`
(placeholder is not a label); icon-conveyed meaning (status dots, badge colors)
needs a text equivalent. React Native: `accessibilityLabel` +
`accessibilityRole` on every `Pressable`/`Touchable*`; group compound
controls with `accessible={true}`; test with TalkBack/VoiceOver, not just
props.

**Touch targets (SC 2.5.8 + platform HIG).** 44×44pt iOS / 48×48dp Android /
24×24px WCAG hard floor, including spacing between adjacent targets. Web
traps: icon buttons at `h-6 w-6` with no padding, tightly stacked row
actions in tables. RN traps: small `Pressable` without `hitSlop`; also check
`allowFontScaling` isn't disabled; font-scaling users are the same audience.

---

## Sweep 3: Interaction & flow catalog

- **The three states.** Every data view needs loading (skeleton > spinner >
  nothing), empty (teaches the first action), and error (cause + retry).
  Most views ship with only the happy one.
- **Forms.** Inline validation on blur (not only on submit); errors adjacent
  to fields; input preserved on failure; submit disabled in-flight with a
  pending label; Enter submits; autofocus on the first field of an
  intentional flow.
- **Destructive actions.** Confirm *or* undo (undo is better); the confirm
  names the object ("Delete *acme-prod*?"); confirm's default focus is Cancel.
- **Navigation.** Current location marked (`aria-current`, active state);
  internal links are `next/link` not full-reload `<a>`; shareable state lives
  in the URL; no dead ends: every screen has an obvious next/back.
- **Latency.** Feedback within ~100ms; pending state past ~400ms; optimistic
  updates where rollback is cheap; no layout shift when async content lands
  (reserve space; `next/image` with dimensions).
- **Responsive/mobile.** Real breakpoint walk at 375px: no horizontal scroll,
  tables degrade deliberately (stack/scroll-with-affordance), fixed elements
  don't cover content or each other, hover-only affordances have a touch
  path. (Deep retrofit → route to mobile-retrofit-for-webapps if installed;
  otherwise log it as one located finding.)
- **Copy.** Buttons say the verb+object ("Save changes", not "OK"/"Submit");
  errors say what happened and what to do; no slop filler in UI copy
  (unslop pairs here).

---

## Per-stack quick greps (draft-finding generators; verify each hit by eye)

**Next.js / React / Tailwind / shadcn:**
```bash
rg -n "outline-none" --glob '*.tsx' | rg -v "focus-visible"   # focus killed
rg -n "<div[^>]*onClick" --glob '*.tsx'                       # fake buttons
rg -n "<img " --glob '*.tsx'                                  # vs next/image + alt
rg -n "text-(gray|zinc|slate|neutral)-(300|400)" --glob '*.tsx'  # contrast suspects
rg -n "href=" --glob '*.tsx' | rg "<a "                       # vs next/link (internal)
fd loading.tsx app/ ; fd error.tsx app/                       # missing-state routes
```

**Expo / React Native:**
```bash
rg -n "(Pressable|TouchableOpacity)" -A3 | rg -v "accessibilityLabel"  # unlabeled
rg -n "allowFontScaling=\{false\}"                                     # scaling off
rg -n "hitSlop" --files-without-match $(rg -l "Pressable")             # small targets
```

**Electron:** menu accelerators per action; full keyboard pass in-window
(Chromium a11y tree via DevTools); focus behavior across multi-window.

Grep hits are *draft* findings. Phase 4 cuts any you can't reproduce in the
running app.
