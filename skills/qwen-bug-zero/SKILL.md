---
name: qwen-bug-zero
description: >-
  Local-Qwen bug pass (token-free). Mechanical form of /bug-zero for a small local model:
  run the bug finders, CONFIRM each finding against the real code before fixing (skip if
  unsure; don't fix phantom bugs), fix one at a time with a regression test under
  build→tests→lint, rescan until two passes find nothing real. Use when you want
  /bug-zero's job done on a local model with no API tokens: hardening a project.
---

# qwen-bug-zero (local correctness)

The Qwen-run version of /bug-zero. The one judgment it must make ("is this finding
real?") is handled by a hard rule: confirm in the code, and when unsure, SKIP (a
phantom bug fixed is worse than one left).

## QWEN LOCAL RULES (full text: ../qwen-forge/references/QWEN-MODE.md)

Requires qwen-forge installed alongside this skill (it holds the shared rules).

- Builder, not architect. ONE change at a time; build → tests → lint; green → commit;
  red → revert, one retry, else STOP. Never weaken tests.
- Unclear / judgment / outward → DECISIONS.md + STOP.
- Local endpoint required: a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama) serving a capable Qwen-class coding model. ONE model loaded, work SEQUENTIALLY, leave memory headroom on unified-memory machines.

## Steps

```
1. SWEEP. Run the finders that fit this project, ONE at a time (sequential, one GPU):
     - the test suite + lint   (real failures; always available, the baseline)
     - ubs / ubs-workflow      (general bugs; if installed)
     - mock-code-finder        (stubs/fakes in live paths; if installed)
     - deadlock-finder-and-fixer (only if there's concurrency; if installed)
   Collect every finding into a list. FIX NOTHING YET.

2. CONFIRM (the important step). For each finding: open the cited file/line and check it
   is a REAL, reachable bug.
     - real      → keep it.
     - not sure  → SKIP it, note in DECISIONS.md. Do NOT fix it.
   Never fix a bug you cannot see in the actual code.

3. FIX. Confirmed bugs, ONE at a time:
     - make the smallest fix.
     - add a regression test that would have caught it.
     - build → tests → lint. green → commit. red → revert, one retry, else STOP.

4. RESCAN. Re-run the finders. Repeat steps 2–4 until two passes in a row find nothing
   real. Then report.
```

## Rules

- Never weaken or delete a test to clear a finding.
- One fix at a time; the regression test per fix is mandatory (it keeps the bug dead).
- If a "bug" is actually a design/scope question → DECISIONS.md, not a fix.

Distilled from /bug-zero. Its ROUTING.md holds the full finder roster and the
smart-model adversarial-verify version for a Claude/human driver.
