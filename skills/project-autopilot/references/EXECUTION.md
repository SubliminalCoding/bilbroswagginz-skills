# Execution: the big-skill pipeline + the backend ladder

How heavy analysis skills get staged, and what actually runs the work. SKILL.md's
EXECUTE step points here when the move is a big skill or when it's time to pick a
backend.

## The Big-Skill Pipeline (routing a heavy analysis/audit skill)

When the chosen move is a **big skill** (an exhaustive analysis/audit that fans
out to subagents: security-sweep, audit-lens, ux-scan,
bug-sweep, optimize-proof sweeps, etc.), do NOT pipe
its output straight to code or beads. Run the canonical staged pipeline, one
timed instruction at a time:

```
1. ORIENT            Read ALL of AGENTS.md + README carefully; deep code
                     investigation of architecture + purpose. (Cold-open deep orient.)
2. APPLY EXHAUSTIVELY Run the big skill super carefully across the WHOLE project.
                     Let it run to completion before reacting to anything.
3. SMART-MODEL REVIEW *Critical, non-skippable.* The top-level model independently
                     re-reviews every top recommendation against the ACTUAL code,
                     because the analysis subagents are often smaller models
                     without full context. The prompt, in substance: review each
                     top recommendation yourself, dig into the actual code, say
                     whether you truly agree, which ones, why or why not, to what
                     extent, and what you think should actually be done.
                     Drop the false positives and over-reach HERE, before they cost
                     a bead or a commit. (This is the heavyweight sibling of LENSES'
                     "re-verify before work.")
4. BEADS             Turn the survivors into beads with the frozen Phase-3a template
                     (see reality-check): self-documenting, deps overlaid.
5. POLISH ×3+        Run the frozen refinement prompt until a pass changes nothing
                     (often 3+ passes). bv to diagnose graph problems.
6. IMPLEMENT         Pick a backend (ladder below) and execute; on bead drain, flip
                     to review + the saturation gate (CONVERGENCE). Same beads on
                     any backend.
```

Steps 4–5 use the **frozen templates** verbatim. Do not paraphrase them (their
exact wording, incl. "DO NOT OVERSIMPLIFY / DO NOT LOSE FEATURES", is load-bearing;
they live in reality-check). The pipeline itself is the
successive-prompter principle in miniature: each step's instruction is issued only
after the previous step finishes.

## Execution backend: detect & degrade (never hard-depend on Codex)

The beads are the same no matter what runs them. The execution backend is a
*runtime detail*, chosen by a cheap capability check at IMPLEMENT time, not a
precondition. When it's time to execute, probe what's actually available and pick
the highest rung that works; degrade silently down the ladder; the floor always
runs. Tell the operator which rung was chosen and why (one line); if you degraded, name
the tradeoff (slower). That's a log note, not a blocker.

```
Ladder (best parallelism → guaranteed floor):
1. ntm + Codex swarm (cod + cc)     IF codex auth is valid AND ntm works. Canonical.
                                     (Verify cod model is newest per swarm rules;
                                      if rate-limited, caam (if installed) can swap accounts.
                                      Tending shape: SAFETY-RAILS "Swarm-mode tending".)
2. ntm + CC-only swarm              Codex gone but ntm works: run cc-only panes.
3. Other-model swarm                gemini via code-review-gemini-swarm-with-ntm
                                     (review, if installed), or local-model agents if you host an
                                     endpoint (e.g. vLLM/Qwen), or an agent runtime
                                     with native parallel sub-agents (a native
                                     delegate-task tool; no external CLI/sub needed).
4. In-process parallel subagents    the Agent/Workflow tool; fan out within THIS
                                     session. Zero external dependency, no sub.
5. Solo CC sequential               THE FLOOR. One batch per tick, green ratchet.
                                     Always available; slower, never blocked.
```

Detection rules:
- Any swarm rung (1–3) operates under `swarm-run`'s two contracts: a sole-writer
  ownership map set up BEFORE panes spawn, and the orchestrator re-running the gate
  itself. A pane's "done, tests pass" is never banked unverified.
- Probe before committing to a rung. Do NOT announce a Codex swarm and then
  discover the sub is dead mid-run. A 5-second auth/health check up front.
- "What to apply AND when" is capability-aware: the same gap routes to the same
  skill (the WHAT), staged at the same time (the WHEN), but EXECUTED on whatever
  backend is live. Missing backend ≠ missing work; it just changes the engine.
- Rungs 3–4 are the answer to "Codex sub may not be valid": another runtime's delegate-task tool
  and the in-process Agent/Workflow tool give real parallelism with no
  subscription at all. Reach for them before assuming you're stuck at solo.
