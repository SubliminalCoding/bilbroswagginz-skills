---
name: qwen-project-autopilot
description: >-
  Local-Qwen 1→done tick loop (token-free). Mechanical checklist form of
  /project-autopilot for a small local model: take the top ready bead, make the smallest
  change, verify build→tests→lint, commit or revert, repeat until the bead graph drains,
  then run qwen-bug-zero and (if speed matters) qwen-perf-max. Stops and asks on any
  judgment call. Use when you want /project-autopilot's job done on a local model with
  no API tokens: pushing a scaffolded project toward done.
---

# qwen-project-autopilot (local 1→done)

The Qwen-run version of /project-autopilot. No lens diagnosis, no operator psychology:
just a tight, gated bead loop. The gates are the safety net.

## QWEN LOCAL RULES (full text: ../qwen-forge/references/QWEN-MODE.md)

Requires qwen-forge installed alongside this skill (it holds the shared rules).

- Builder, not architect. Judgment calls belong to the user.
- ONE change at a time; build → tests → lint after each; green → commit; red → revert,
  one retry, else STOP. Never weaken tests.
- Build ONLY what the Intent needs; extras → SOMEDAY.md.
- Unclear / judgment / outward → DECISIONS.md + STOP.
- Local endpoint required: a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama) serving a capable Qwen-class coding model. ONE model loaded, work SEQUENTIALLY, leave memory headroom on unified-memory machines.

## Before the loop

1. Confirm the baseline is green (build + tests). If red → fixing the baseline is the
   first task; do that under the loop's gates before anything else.

## THE LOOP (repeat)

```
a. `br ready` → take the TOP task. If none → go to "When the beads drain".
b. Make the SMALLEST change that completes ONLY that task. Touch as little as possible.
c. Run build → tests → lint.
     green → commit (short msg), `br update <id> --status closed`, report one line.
     red   → `git restore` the change; try ONE more fix; still red → STOP and ask.
d. If the task needs a design / naming / scope / taste decision → STOP, write it to
   .autopilot/DECISIONS.md, move to the next ready task.
e. Go to (a).
```

## When the beads drain (`br ready` empty)

```
1. Run qwen-bug-zero (find + fix real bugs).
2. If the Intent cares about speed AND features are settled → run qwen-perf-max.
3. Report: "intent features built, bugs swept" + hand to the user for ship (qwen-ship-it).
```

## Rules that keep it on track

- Never widen scope. New ideas that come up → SOMEDAY.md, not new beads, unless the user says so.
- Small batches only: one bead, one verify, one commit. The green ratchet is the whole
  safety system here (there is no smart-model review on a local-only run).

Distilled from /project-autopilot. Its references hold the full
lens/routing/operator-moves version for a Claude or human driver.
