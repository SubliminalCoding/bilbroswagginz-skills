---
name: qwen-ux-maximizer
description: >-
  Local-Qwen UI/UX pass (token-free), heavily user-gated because taste is not a small-
  model job. Mechanical form of /ux-maximizer: fix only CONCRETE, checkable UI issues
  (contrast, missing labels, broken layout, errorless failures) under the gates; for
  anything about look/feel/"is this premium", screenshot it and STOP for the user. Use
  when you want /ux-maximizer's job done on a local model with no API tokens: basic UI
  hardening.
---

# qwen-ux-maximizer (local UI/UX; taste stays with the user)

The Qwen-run version of /ux-maximizer, with an honest limit: a small local model cannot
judge taste. So it fixes only the OBJECTIVE issues and hands everything subjective to the user.

## QWEN LOCAL RULES (full text: ../qwen-forge/references/QWEN-MODE.md)

Requires qwen-forge installed alongside this skill (it holds the shared rules).

- Builder, not architect. ONE change at a time; build → tests → lint; green → commit;
  red → revert, one retry, else STOP. Never weaken tests.
- Unclear / judgment / outward → DECISIONS.md + STOP.
- Local endpoint required: a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama) serving a capable Qwen-class coding model. ONE model loaded, work SEQUENTIALLY, leave memory headroom on unified-memory machines.

## Steps

```
1. CHECKABLE FIXES ONLY. Run ux-scan / accessibility checks and fix issues that are
   OBJECTIVE and verifiable, one at a time under the gates:
     - color contrast below standard
     - missing alt text / aria-labels / form labels
     - broken or overflowing layout, off-screen elements
     - actions that fail with no message to the user
     - obvious mobile breakage (unreachable buttons, tables overflowing)

2. TASTE → USER. Anything about look, feel, "make it premium", style, palette, vibe,
   layout aesthetics → take a screenshot, write what you'd change in DECISIONS.md, and
   STOP. Do NOT invent a design direction. You cannot judge whether it looks good.

3. NEVER run idea-wizard locally to brainstorm a redesign: overbuild risk, and it's
   the user's creative call.
```

## Rule

- The hard line: objective accessibility/correctness UI bugs = you fix them. Subjective
  beauty/taste = screenshot + STOP. When unsure which side a thing is on, treat it as taste.

Distilled from /ux-maximizer. Its PROMPTS.md holds the full wow-factor idea-wizard
pipeline for when the user or Claude is driving the design.
