---
name: skillify
description: >-
  Turn a hard-won solution into a reusable, auto-wired skill the moment you solve
  something tough, then register it across every runtime you use so every agent
  you run applies it automatically instead of re-deriving it. Use when you just
  cracked a non-trivial problem whose METHOD generalizes (a debugging
  breakthrough, an architecture pivot, a gotcha that ate hours, a workaround for
  an external system's quirk, a hardening pattern), or when you say "skillify
  this", "make a skill out of this", "capture this", "remember how we did this",
  "bake this in". This is also a STANDING HABIT (add it to your global
  CLAUDE.md): after solving a generalizable problem, run it proactively.
---

# skillify

> **Core insight:** A tough problem solved once should be solved once. Capture
> the *method* (not the incident), wire it into every runtime, and make it
> auto-surface, so future-you and the other agents never re-derive it.

## The trigger (when to run; this is the automatic part)

Run skillify right after solving a problem where **the solution generalizes**.
The test: *"Would future-me, or another agent runtime, on a DIFFERENT project benefit from
this?"*

| Skillify it | Don't (use the project runbook/lessons instead) |
|-------------|--------------------------------------------------|
| A debugging method that transfers | A one-off fix specific to one file/bug |
| An architecture pivot / pattern | A config value or a typo |
| A gotcha that cost real time | Something the code already documents |
| A workaround for an external system's quirk | A decision only this project cares about |
| A hardening / safety / monitoring pattern | |

When in doubt and it's borderline-specific, write a CLAUDE.md lesson, not a skill.
Don't clutter the registry: a skill that's too narrow never fires.

## The process

### 1. Extract the METHOD, not the incident
Strip the project specifics. Keep the transferable decision rules, the gotchas,
the anti-patterns. The skill reads like a **playbook**, not a postmortem. ("When
X, do Y because Z; watch for gotcha W", not "on 2026-06-05 we hit...").

### 2. Write the SKILL.md in the house format
`~/.claude/skills/<kebab-name>/SKILL.md`:
- Frontmatter `name` + `description`. The **description is load-bearing**: it's
  what makes the skill auto-surface, so list the concrete SYMPTOMS and PHRASES a
  future session would say ("the token keeps expiring", "make this a fortress").
  Vague description = never fires.
- Body: **Core insight** (one line) → the decision/process → a gotchas /
  anti-patterns table → a checklist. Tight: ~80–130 lines. Reference sibling
  skills by name. Match the tone of existing `~/.claude/skills/*/SKILL.md`.
- House rules: no em dashes in any output; concrete over abstract.

### 3. Choose scope FIRST, then register (this is what makes it STICK)

**DEFAULT to PROJECT scope.** Most skills are learnings about one app's quirks
(its detection, outreach, deploy, auth). Write them to
`<project>/.claude/skills/<name>/SKILL.md` so they only load in that project and
don't clutter every global session. Reference them from that project's CLAUDE.md.
Do NOT global-register or broadcast project-specific skills (standing rule).

**Promote to GLOBAL only for genuinely cross-project tooling**: meta/reasoning
skills (like skillify, problem-breakthrough) that apply on ANY project. For those,
run your registration script if you have one (the name below is an example):
```bash
~/bin/skill-register <kebab-name> [category]   # symlink into your other runtimes + auto-memory + reindex
```
It symlinks into your other agent runtimes' skill directories (symlink never copy), adds
an auto-memory pointer, and re-indexes your notes/search index if you keep one. Without
a script, do those steps by hand. THEN, for a global skill, you
still: add a `**Last updated:**` entry atop your shared cross-agent memory file
(prose: what it is + a note that your other runtimes apply it when work matches), and if broadly
applicable, add it to the "Standing skill directives" section of your global CLAUDE.md.

For a PROJECT-scoped skill: skip registration / cross-runtime symlinks / shared memory entirely;
just add the lesson + skill reference to that project's CLAUDE.md.

### 4. Verify the wiring
Confirm the skill resolves in EVERY runtime you registered
(e.g. `~/.claude/skills/<name>/SKILL.md` and each other runtime's skill directory).
A registration script should print this check for you.

## Make future application automatic
The skill only pays off if it FIRES later. Two levers:
- **Description quality** (step 2): the matcher keys off it.
- **The standing habit** in your global CLAUDE.md: a "Standing skill directives" section lists the
  high-value skills to ALWAYS apply when work matches, and says to skillify new
  wins. When you create a broadly-applicable skill, add it to that list too.

## Anti-patterns

| Don't | Do |
|-------|-----|
| Skillify a one-off | Project runbook lesson instead |
| Write a postmortem | Write a playbook (rule, not story) |
| Vague description | List concrete trigger phrases/symptoms |
| Copy across runtimes | Symlink (never copy), by script or by hand |
| Create it and stop | Register in all places + verify every runtime |
| Overbuild a 400-line skill | Tight playbook; small models + matchers prefer lean |

## Checklist
- [ ] Solution genuinely generalizes (passed the "different project" test)
- [ ] Method extracted, not the incident
- [ ] SKILL.md written, description rich with trigger phrases, tight + house-format
- [ ] Registered globally only if cross-project (cross-runtime symlink + auto-memory + reindex)
- [ ] Shared cross-agent memory entry added (other-runtimes directive)
- [ ] Project runbook updated if project-specific
- [ ] Added to your global CLAUDE.md "Standing skill directives" if broadly applicable
- [ ] Resolves in every registered runtime
