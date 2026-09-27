# QWEN-MODE: operating doctrine for the local-Qwen family

The full rules behind the condensed block embedded in each qwen-* skill. These exist
because a small local model executes narrow, literal, well-gated tasks well and does
open-ended judgment/orchestration poorly. So the deterministic gates carry the weight,
and judgment is externalized to the user (or a hybrid Claude controller).

Every sibling qwen-* skill points here as `../qwen-forge/references/QWEN-MODE.md`, so
qwen-forge must be installed alongside them.

## The hard rules (every qwen-* skill obeys these)

1. **You are a builder, not an architect.** Do exactly what the task says. Judgment
   calls (design, naming a big choice, scope, tradeoffs, taste) belong to the user.
2. **One small change at a time.** Never batch.
3. **Verify by command after EVERY change, in order: build → tests → lint.**
   - all green → commit (short message) + report one line.
   - any red → `git restore`/revert the change; try ONE more fix; still red → STOP and ask.
   - never delete, skip, or weaken a test to go green.
4. **Build only what the Intent needs.** Any extra idea → write to `SOMEDAY.md`, do not build.
5. **Stop, don't guess.** Unclear task, ambiguity, or any judgment call → write the
   question to `<project>/.autopilot/DECISIONS.md` and STOP.
6. **Never do anything outward or irreversible** (push, publish, deploy, delete data,
   touch secrets) → DECISIONS.md + STOP for the user.

## Local execution (requires a local model host, e.g. a DGX Spark / GB10-class box)

- Runs against a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama)
  serving a capable Qwen-class coding model. Select it via whatever model-switch helper
  your local-agent CLI provides. No local endpoint → the qwen-* family doesn't apply;
  use the Claude/Codex forms instead.
- ONE model on the GPU at a time → work strictly **SEQUENTIALLY**. No swarm, no parallel
  local agents.
- Leave memory headroom on unified-memory hardware (with vLLM, for example,
  `--gpu-memory-utilization` ≤ 0.75).
- enable_thinking:false for routine edits (faster) where the model supports it; thinking
  ON for a real planning step.

## What the small model CANNOT do (so it's externalized)

- Ideation / taste / "is this premium" / "is this done" → the user, or a short Claude session.
- Lens diagnosis / smart-model self-review of findings → replaced by the deterministic
  gates (build/test/lint + revert-on-red) and the stop-and-ask reflex.

## Best quality, still near token-free: HYBRID

A short Claude/Codex session does the planning, review and taste (a little spend, the
thinking); Qwen grinds the beads (free, local, the volume). This is the execution ladder's
local-model rung: smart controller, free local muscle. The fully-local qwen-* family is
the zero-spend option; hybrid is the quality option.
