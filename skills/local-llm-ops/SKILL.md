---
name: local-llm-ops
description: Operating discipline for local LLM hosting on unified-memory hardware (NVIDIA GB10 / DGX Spark class). Use when configuring Ollama or vLLM, diagnosing slow inference, deciding between local and hosted model backends, debugging "model not loaded" errors, tuning GPU memory utilization, or setting up tool-calling with Qwen/Llama-class models.
---

# Local LLM Ops

Running local models on unified-memory hardware is not the same as running them on a box with dedicated VRAM. Most tuning advice online assumes a discrete GPU. Apply it on a GB10-class machine and you will silently steal system RAM from every other process on the box. This skill encodes the rules that keep local inference fast and the rest of the machine alive.

## The hardware assumption

This skill assumes the host is a unified-memory box: CPU and GPU share the same physical RAM pool. Concretely, an NVIDIA GB10 / DGX Spark with 128GB shared, but the rules generalize to any Apple Silicon or Grace Hopper-class system. On a box with a discrete RTX card and a separate CPU DIMM pool, the gpu-memory-utilization rule below is wrong and should be inverted.

If you don't know which category the box is in, check before tuning.

## Hard rules

1. **One Ollama model loaded at a time.** Period. Loading a second model does not parallelize; it swaps. On GB10, the cost of a warm swap between models is the difference between ~0.6s and ~16s response time. If a workflow needs two models, it needs two serial phases, not concurrent loads.

2. **`--gpu-memory-utilization` stays ≤ 0.75 on unified memory.** Going higher does not give the model more VRAM. It takes that memory out of the same pool the OS, your vector DB, and every other process share. Once system RAM is starved, the kernel starts swapping, and everything including inference collapses. The 0.75 ceiling is not a default to "tune up later"; it's a ceiling.

3. **Verify what actually loaded.** The `--model` flag says what you asked for. The served model is what was actually loaded. After any vLLM launch, confirm:
   ```bash
   curl -s localhost:8000/v1/models | jq '.data[0].root'
   ```
   The HuggingFace path that comes back is the source of truth. Expected for this stack: the FP8 Qwen3.6 artifact, not any Qwen3-VL or older Qwen3 variant that may still be cached from an earlier attempt.

4. **Prefix caching is mandatory for tool-using workflows.** Without `--enable-prefix-caching`, every tool call reprocesses the full system prompt and tool definitions. With it, the KV cache carries across turns and tool-calling latency drops by a large constant factor. There is no workload this flag hurts.

5. **Tool-call and reasoning parsers must match the model.** For Qwen3/3.6-class models, the required flags are `--tool-call-parser qwen3_xml --reasoning-parser qwen3`. Omitting either produces tool calls that look correct in raw output but fail to parse on the client side. It is a silent failure mode that wastes a lot of debugging time if you don't know to check.

## The default vLLM launch shape

For Qwen3.6-35B-A3B-FP8 on this hardware, the baseline command is:

```bash
vllm serve <model-path> \
  --host 0.0.0.0 --port 8000 \
  --gpu-memory-utilization 0.75 \
  --tool-call-parser qwen3_xml \
  --reasoning-parser qwen3 \
  --enable-prefix-caching
```

Changes to this shape require a reason. "I saw a blog post recommend X" is not a reason. A measured regression on a specific workload is a reason.

## When diagnosing slow inference

Work the list in order. Do not skip.

1. **Is a second model loaded?** Check `ollama ps`. If more than one, unload the one not in use.
2. **Is the system swapping?** Check `free -h` and `vmstat 1`. If swap is non-zero and growing, gpu-memory-utilization is too high. Lower it.
3. **Is the right model actually loaded?** `curl localhost:8000/v1/models | jq`. If the served model is not what the launch command specified, the cache served an older artifact. Clear or pin.
4. **Is prefix caching on?** Check the vLLM startup logs for the flag. If missing, every tool turn is re-tokenizing the system prompt.
5. **Is another service competing for GPU?** Ollama and vLLM on the same box, both with GPU claims, will fight. One or the other at a time during heavy workloads.

## Local vs hosted: the decision rule

Local is the default. Hosted (Anthropic, OpenAI, etc.) is the fallback, invoked for specific reasons:

- **Use local when:** the workload is high-volume, the data is sensitive, the latency budget is tight and the network round-trip matters, or the cost-per-token of hosted would exceed the cost-of-electricity for local.
- **Use hosted when:** the task requires a capability the local model genuinely cannot do (e.g., a specific tool-use pattern or a context length beyond what the local model supports), or the task is one-shot and not worth the context-switch of loading a local model, or the task is for a client who is paying for hosted.

The decision must be named per task, not set globally. Defaulting to hosted "to be safe" is how Anthropic bills add up without anyone noticing.

## Dual-mode clients (local ↔ hosted switching)

If your agent client switches between a local profile (e.g. `vllm/qwen3.6`) and a hosted one (e.g. `anthropic/claude-opus-4-6`), two things are easy to get wrong:

1. **The auth key lives in two places.** The env var and the profile config. The profile overrides the env when both are set. If a switch isn't taking effect, check which layer is actually winning.
2. **Don't switch profiles mid-session without also restarting the client.** Some clients cache the model metadata and will keep sending Anthropic-shaped requests to the local endpoint (or vice versa) until restart.

## When this skill should say no

- **Before raising `--gpu-memory-utilization` above 0.75 on unified memory.** Stop. Find the real bottleneck first.
- **Before loading a second Ollama model to "try them side by side."** Stop. Run them serially and record the outputs.
- **Before adding a new vLLM flag from a blog post without a measured baseline.** Stop. Run the baseline command, get a number, then change one flag at a time.
- **Before switching a production workflow from local to hosted because it feels slow.** Stop. Work the diagnosis list first. Most "local is slow" is actually "two models are loaded" or "prefix caching is off."
