---
name: ux-maximizer
description: >-
  Exhaustively maximize a project's UI/UX to premium, "wow-factor" quality. A staged
  successive-prompter: idea-forge for a premium vision → implement the top ideas →
  a cascade of UX audits, UI polishes, and design-intel passes → repeat until the
  passes stop finding material improvements. Use when "make this premium", "wow
  factor", "maximize the UI/UX", "design pass", "make it world-class", or elevating
  a functional-but-plain app into something that makes users go wow.
---

# UX Maximizer

Take a project that *works* and grind it until it *feels premium*. Not one polish
pass: a vision, implemented, then ground down through repeated audit → polish →
design-intelligence cycles until the passes stop finding anything worth changing.

> **Core Insight:** Taste is a process, not a flash. A premium feel doesn't come
> from a single pass; it comes from a bold vision (idea-forge) implemented, then
> sanded smooth by repeated cycles. Like its sibling **project-autopilot**, this is
> a *successive-prompter*: each stage's instruction fires only when the prior stage
> finishes (one thing at a time, done well), and a **green ratchet** means the app
> never breaks while it's being beautified. Design *taste* stays the user's. The skill
> does the work, the user's eyes sign off.

## Preconditions / guards (check before Phase 1)

1. **The app works.** UI/UX maximization is a *maturity activity*: you beautify a
   working thing, not a broken one. If core function is broken, fix that first.
2. **Green baseline + checkpoint.** The ratchet needs a recoverable starting point.
3. **⚠️ Late-stage ideation guard (standing rule):** the Phase-1
   idea-forge run is a *mass-dispatch* that can destabilize a mature project. If the
   project is late-stage (**>10 modified files OR >3 in-flight beads**), STOP and
   confirm with the user before ideating.

## The Pipeline (staged: issue each prompt only when the prior finishes)

### Phase 0: Orient
Read the project. Understand three things: *what it is*, *who the user is*, and the
*current UI/UX state* (screens, flows, components, the existing design language).
Checkpoint a green baseline.

### Phase 1: WOW vision (idea-forge)
Run `idea-forge` with the verbatim creative-unlock prompt (it's calibrated to
break the model out of trained incrementalism; see references/PROMPTS.md). Rank the
output; **take the top 5 recommendations.**

### Phase 2: Implement the top 5
Operationalize the 5 into beads (beadsmith), then implement them **one at a
time**, each under the green ratchet with its own checkpoint. Design taste is the user's:
after each (and especially for a bold direction change), **show the running result /
screenshots** and let the user gut-check before it's banked. A redesign direction is an
ask-user decision, a routine refinement is auto.

### Phase 3: The quality cascade (the calibrated counts)
Once the 5 are in, run the cascade in this order, each via `deepen`
with progressive deepening, every pass's findings fixed under the green ratchet:
- **3× `ux-scan`:** heuristic + accessibility evaluation → fix findings
- **5× `polish-pass`:** iterative visual elevation toward Stripe-level quality
- **3× `design-intel`:** design-intelligence passes (styles, palettes, type, layout)

(5 / 3 / 5 / 3 are the calibrated starting counts, tunable per project.)

### Phase 4: Exhaust until complete
Loop Phase 3 until **saturation**: passes stop surfacing material improvements
(diminishing returns, not literal zero). If a pass reveals a genuinely new premium
*direction*, re-ideate (back to Phase 1, guard re-checked) rather than grinding the
old one. Then surface the before/after to the user for final taste sign-off (the user's QA
move) and write a short done report.

## Verify by looking (not just by passing)

UI/UX cannot be proven by a green test suite; it has to be *seen*. After passes,
actually run the app and capture what it looks like (use /run or /verify if installed, or
start it yourself; for webapps, screenshot the key flows). Tests prove it still works; only eyes prove it
looks premium. The user is the final visual QA.

## Shared mechanisms (inherited from project-autopilot)

Green ratchet (every change ends build+tests+smoke green or reverts), decision inbox
(taste/direction calls queue for the user), saturation-based convergence, and
capability-aware execution (solo by default; ntm swarm optional when installed) all work as they do in
project-autopilot; see that skill's CONVERGENCE.md / SAFETY-RAILS.md / EXECUTION.md
rather than duplicating them here. **Integration:** project-autopilot's UX lens can
route a project here for the exhaustive treatment.

## Skills it orchestrates

idea-forge · ux-scan · polish-pass · design-intel · deepen · beadsmith ·
unslop (UI copy). Optional, if installed (otherwise the core skills above cover
the work): premium-palette-design (palette work) ·
interactive-visualization-creator (data viz) · mobile-retrofit-for-webapps (mobile
gaps) · run / verify (look at the result).

## THE EXACT PROMPT

```
/ux-maximizer
Target: <project path>
Counts: <ideas/ux-scans/polishes/design-intel; default 5/3/5/3>
Mode: solo (default) | swarm (optional; needs ntm installed)
```

## Anti-Patterns

- One polish pass and calling it "premium". Taste is a process, not a flash.
- Running the Phase-1 idea-forge mass-dispatch on a late-stage project without
  asking (the guard exists for exactly this).
- Beautifying a broken app. Function first, beauty second.
- Auto-banking a bold redesign direction without the user's taste gut-check.
- Trusting "tests pass" as proof it looks good. You have to look at it.
- Grinding the same cascade past saturation instead of re-ideating a fresh direction.

## References

- [PROMPTS.md](references/PROMPTS.md): the verbatim wow-prompt for idea-forge + cascade spec
