---
name: qwen-forge
description: >-
  Local-Qwen lifecycle conductor (token-free; requires a local OpenAI-compatible model
  endpoint such as vLLM, SGLang, llama.cpp or Ollama). Detects the project's stage with
  simple file checks and runs the matching qwen-* skill (qwen-kickstart →
  qwen-project-autopilot → qwen-ship-it) sequentially on one GPU. The mechanical,
  checklist form of /forge for a small local model. Use when you want /forge's job done
  on a local model with no API tokens, running the full build lifecycle at zero token cost.
---

# qwen-forge (local conductor)

The Qwen-run version of /forge. Same arc, mechanical form: detect the stage by concrete
checks (not judgment), run the matching qwen-* skill, advance. One GPU, so sequential.

## QWEN LOCAL RULES (obey every step; full text: references/QWEN-MODE.md)

- You are a builder, not an architect. Judgment calls belong to the user.
- ONE small change at a time. After each: build → tests → lint. Green → commit + report.
  Red → revert, one retry, else STOP and ask. Never weaken tests.
- Build ONLY what the Intent needs; extra ideas → SOMEDAY.md.
- Unclear / judgment call / outward-or-irreversible → write to .autopilot/DECISIONS.md and STOP.
- Local endpoint required: a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama) serving a capable Qwen-class coding model. ONE model loaded, work SEQUENTIALLY, leave memory headroom on unified-memory machines.

## Detect the stage (simple checks, in order)

```
1. Empty repo / no source / no build file (no package.json, Cargo.toml, etc.)
        → run qwen-kickstart   (0→1: scaffold it)
2. It builds and has tests, but features are incomplete vs the Intent
        → run qwen-project-autopilot   (1→done)
3. It builds, tests pass, and it delivers the Intent
        → run qwen-ship-it   (pre-flight only; buttons belong to the user)
4. Already shipped → STOP, report.
```

## Loop

```
a. Detect the stage (above).
b. Run that one qwen-* skill; let it finish its own loop.
c. When it reports done, re-detect and go to the next stage.
d. STOP for the user at: any ship button, any DECISIONS.md item, or anything you're unsure about.
```

## First steps

1. Read README.md + AGENTS.md fully; say back in 3 bullets what the project is.
2. Make sure it builds + tests pass right now (baseline). If not and a scaffold exists,
   that's qwen-project-autopilot's first job: fix the baseline before new work.
3. Detect the stage and begin.

## Notes

- This is the zero-token local option. For higher quality at near-zero spend, run HYBRID
  (a short Claude session plans/reviews; Qwen executes). See references/QWEN-MODE.md.
- Distilled from /forge. That skill (and project-autopilot's references) hold the full
  judgment-rich version if a human or Claude is driving instead.
