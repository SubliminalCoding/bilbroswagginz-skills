---
name: qwen-ship-it
description: >-
  Local-Qwen release pre-flight (token-free). Mechanical form of /ship-it: do ONLY the
  reversible pre-flight (confirm green + intent, bump version, changelog, build artifacts
  + checksums, verify installer), then list every outward button (push/publish/deploy/
  submit) with its rollback in DECISIONS.md and PRESS NOTHING. Use when you want
  /ship-it's job done on a local model with no API tokens: preparing a release while the
  buttons stay with the user.
---

# qwen-ship-it (local release pre-flight)

The Qwen-run version of /ship-it. The whole skill is "do the safe pre-flight, then STOP
at every button." A small local model must NEVER press an outward action.

## QWEN LOCAL RULES (full text: ../qwen-forge/references/QWEN-MODE.md)

Requires qwen-forge installed alongside this skill (it holds the shared rules).

- Builder, not architect. ONE change at a time; build → tests → lint; green → commit;
  red → revert, one retry, else STOP. Never weaken tests.
- Local endpoint required: a local OpenAI-compatible endpoint (vLLM, SGLang, llama.cpp or Ollama) serving a capable Qwen-class coding model. ONE model loaded, work SEQUENTIALLY, leave memory headroom on unified-memory machines.
- ABSOLUTE: never push, publish, deploy, submit, or run cargo/npm publish. Those are buttons.

## Steps (reversible pre-flight only)

```
1. GATE. Confirm build + tests are green AND the project delivers its Intent. If not →
   STOP, report what's missing. Do not prepare a release of something broken/incomplete.

2. SCAN. Check the artifacts/config for secrets/keys. Found one → STOP, DECISIONS.md.

3. PREP (all reversible, commit as you go):
     - bump the version (semver).
     - write the changelog / release notes.
     - build the release artifacts + checksums locally.
     - verify the installer actually installs/runs, if there is one.
     - deploy to a PREVIEW/staging env only, never production.

4. BUTTONS. Write to .autopilot/DECISIONS.md every outward action, each with its exact
   command and its rollback:
     - push tag / release · publish GH release · promote preview→production ·
       submit to store · cargo/npm publish
   PRESS NONE OF THEM. Report the ready package + the button list to the user.
```

## Rule

- Everything reversible and local = you do it. Everything outward/irreversible = a button
  in DECISIONS.md for the user. When unsure which, it's a button.

Distilled from /ship-it. Its ROUTING.md holds the full button-discipline + platform-
decoupling version for a Claude/human driver.
