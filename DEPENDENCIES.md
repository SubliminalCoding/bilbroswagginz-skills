# Dependencies

What this pack actually needs on a machine, and what it merely uses when present.

## Required: beads (`br` + `bv`)

The one hard dependency. Treat it like git. beads is the open-source beads_rust
issue tracker: `br` manages the project's issue graph, `bv` triages it. The
pack's operational skills (idea-forge, beadsmith, project-autopilot, kickstart,
bug-sweep, the forge lifecycle) plan and track work by creating and validating
`br` task graphs. Without it they have nowhere to put the work.

Install beads_rust for your platform so that `br list` and `bv` run; see the
beads_rust project README. Skills that only read and report (reality-check)
degrade gracefully when `br` is missing and say so in their output, but
anything that operationalizes work needs it.

## Optional: skills degrade gracefully without these

- **`ntm` + `vibing-with-ntm`**: parallel multi-agent swarm execution. Every
  skill that offers `Mode: swarm` also runs solo, and solo is the default;
  dueling-idea-forge runs its duel as sequential isolated sub-agent invocations
  when `ntm` is absent. A missing swarm tool never blocks a run; it only slows
  it.
- **Local-model infra (Ollama, vLLM, SGLang or llama.cpp)**: needed only where a skill explicitly
  offers a local path: the qwen-* family, local-llm-ops, forge's KICKOFF-QWEN
  variant, and the execution ladder's local-model rung. No local endpoint → use
  the Claude/Codex forms; nothing else in the pack assumes one is running.
- **Platform/build helpers (`rch`, `dsr`, `caam`, `memguard`)**: remote-build,
  release, account-swap, and memory-guard conveniences. Each appears only as an
  "if installed" rung with a standard fallback: your normal CI / gh-actions /
  local builds, and bare launches on machines that aren't unified-memory boxes.

## Composes with, does not bundle

Standard tool-wrapper skills (vercel, wrangler, gcloud, the gh-* family,
supabase, the testing-* family, deadlock-finder-and-fixer, mock-code-finder,
ubs and similar) are optional integrations. The pack routes to them when
they're installed and routes around them when they're not. They are not shipped
here and not required.
