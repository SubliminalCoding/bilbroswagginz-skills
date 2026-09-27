---
name: qwen-kickstart
description: >-
  Local-Qwen 0→1 scaffolder (token-free). Mechanical checklist form of /kickstart for a
  small local model: write the Intent, pick the SIMPLEST stack, build a walking skeleton
  that runs end-to-end, list only the features the Intent needs as beads, hand off to
  qwen-project-autopilot. Anti-overbuild by construction. Use when you want /kickstart's
  job done on a local model with no API tokens: starting a new project.
---

# qwen-kickstart (local 0→1)

The Qwen-run version of /kickstart. Build the lean scaffold and hand off. Do NOT try to
finish the whole project here.

## QWEN LOCAL RULES (full text: ../qwen-forge/references/QWEN-MODE.md)

Requires qwen-forge installed alongside this skill (it holds the shared rules).

- Builder, not architect. Judgment calls belong to the user.
- ONE change at a time; build → tests → lint after each; green → commit; red → revert,
  one retry, else STOP. Never weaken tests.
- Build ONLY what the Intent needs; extras → SOMEDAY.md.
- Unclear / judgment / outward → DECISIONS.md + STOP.
- Local endpoint required: a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama) serving a capable Qwen-class coding model. ONE model loaded, work SEQUENTIALLY, leave memory headroom on unified-memory machines.

## Steps (in order)

```
1. INTENT. Write one paragraph: who it's for, the one job, what "done" looks like, keep
   it small. Show the user, wait for "go". This is the CEILING: never build past it.

2. STACK. Pick the SIMPLEST, most boring, most proven way to build it. If the choice is
   big or non-obvious (core framework, data model) → STOP, ask in DECISIONS.md. Default
   to the obvious lean option; do not over-engineer.

3. WALKING SKELETON. Build the THINNEST thing that runs end-to-end (one real path through
   the whole app) + ONE passing test. Commit. This is the baseline the ratchet needs.

4. BEADS. List the features the Intent needs as beads (`br`). One bead per feature. Add
   NOTHING beyond the Intent (extras → SOMEDAY.md).

5. HAND OFF. Report: "scaffold green, beads ready" and hand to qwen-project-autopilot.
```

## Anti-overbuild (these are the point; do not skip)

- Lean default in step 2; walking skeleton before any deep build in step 3.
- Intent is the ceiling; beyond-Intent ideas go to SOMEDAY.md, never built.
- No abstraction/framework/config layer until 3 real places actually need it.
- Do NOT run idea-wizard locally to brainstorm many features; that's the user's call
  (overbuild risk on a small model). Build the obvious Intent features only.

Distilled from /kickstart (its references hold the full idea-wizard / advisor-council
version for when a human or Claude is driving the 0→1 design).
