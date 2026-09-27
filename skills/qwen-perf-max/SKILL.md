---
name: qwen-perf-max
description: >-
  Local-Qwen performance pass (token-free). Mechanical form of /perf-max for a small
  local model: only on settled code with a real workload, profile to a scored hotspot
  list, optimize the top one at a time, prove behavior is unchanged AND measure the win
  (revert if no measurable gain or behavior changed), re-profile until the wins dry up.
  Use when you want /perf-max's job done on a local model with no API tokens: speeding
  up a project.
---

# qwen-perf-max (local performance)

The Qwen-run version of /perf-max. Profile first, always. Never optimize by guess.

## QWEN LOCAL RULES (full text: ../qwen-forge/references/QWEN-MODE.md)

Requires qwen-forge installed alongside this skill (it holds the shared rules).

- Builder, not architect. ONE change at a time; build → tests → lint; green → commit;
  red → revert, one retry, else STOP. Never weaken tests.
- Unclear / judgment / outward → DECISIONS.md + STOP.
- Local endpoint required: a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama) serving a capable Qwen-class coding model. ONE model loaded, work SEQUENTIALLY, leave memory headroom on unified-memory machines.

## Gate (do NOT start unless both are true)

1. The Intent actually cares about speed.
2. Features are settled (not still being built). Optimizing churning code is wasted.

If unsure whether to run this → ask the user (DECISIONS.md).

## Steps

```
1. WORKLOAD. There must be a realistic way to measure (a benchmark / repeatable run). If
   there isn't one → STOP and make a small one first; you cannot optimize what you can't
   measure.

2. PROFILE. Run hotspot-scout against the workload → a SCORED hotspot
   list (CPU / memory / I/O). No profile output → do NOT optimize.

3. OPTIMIZE. The TOP hotspot, ONE change at a time:
     - make the change.
     - tests green (behavior UNCHANGED); golden/metamorphic test if outputs matter.
     - measure before/after (the number from step 1's workload).
     - NO measurable win, OR behavior changed → REVERT. Otherwise commit.

4. RE-PROFILE. The bottleneck moves after each fix. Re-run step 2. Stop when a pass finds
   no material win left. Report the before/after numbers.
```

## Rules

- Never keep a "faster" change that altered behavior or reds a test (fast-and-wrong = wrong).
- Never keep a change with no measured win (that's just added complexity).
- Profile every round; do not optimize the same spot twice without re-measuring.

Distilled from /perf-max. Its ROUTING.md holds the full version for a Claude/human driver.
