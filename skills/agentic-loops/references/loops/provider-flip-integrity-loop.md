---
name: provider-flip-integrity-loop
goal: Prove every model-flip helper actually points the runtime at the intended backend and that tokens genuinely resynced
cadence: on-demand (after any flip) + weekly audit
run_via: /loop
routes_to: [local-llm-ops, trace-it]
stop_condition: Probe-reported model == requested model for every configured profile, no stale-token warnings, on two consecutive verifications
verification_gate: '"Flipped" requires the LIVE PROBE''s model identity to match the request; config files saying the right thing is not enough'
---

# Provider-Flip Integrity Loop

**Goal.** Multi-runtime setups that flip between local backends (vLLM, Ollama) and
hosted providers have a documented trap: the CLI lies about what it's using: the
config names one model while the runtime quietly serves another, and refreshed OAuth
tokens never actually get pushed. This turns flip-and-pray into a probe-verified gate.

**Each iteration.**
1. Run the flip (your `<runtime>-flip <profile>` helper) then its `status` / `verify` step.
2. Send a one-shot probe through the runtime and assert the responding model label matches the requested target (local server vs hosted provider), with thinking/reasoning modes disabled where chat-speed matters.
3. For OAuth-based flips, confirm the refreshed token was actually pushed; if not, apply the documented manual resync.

**Stop when** the probe-reported model equals the requested model for every configured profile, with no stale-token warnings, on two consecutive verifications.

**Verification gate.** "Flipped" requires the live probe's model identity to match the request. Config files claiming the right model is NOT enough: configs routinely say one thing while the CLI serves another, and tokens silently fail to push. A label mismatch is FAILED.

## Copy-paste prompt
```
/loop Verify model flips actually took. Each pass: run the flip helper and its
status/verify step, then send a one-shot probe through the runtime and assert the
RESPONDING model label matches the requested target (local vLLM/Ollama server vs
hosted provider), with thinking mode disabled where chat speed matters. For
OAuth-based flips, confirm the refreshed token was actually pushed and apply the
manual resync if not. A config file claiming the right model is NOT proof; only
the live probe is. Stop when every profile's probe model == requested model with no
stale-token warnings on two consecutive verifications.
```

**Cadence / notes.** Run after every flip; weekly audit otherwise. Even a pinned config can silently fall back, so always verify the live label. Route root-cause digs to `trace-it`.
