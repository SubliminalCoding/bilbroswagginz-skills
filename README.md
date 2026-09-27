# BilbroSwagginz Skills

52 opinionated [Claude Code](https://docs.claude.com/en/docs/claude-code) skills for taking software from rough idea to shipped release: planning, building, bug hunts, performance, UX, writing and release prep. Free and open source under the [MIT license](LICENSE).

Browse every skill with examples at [bilbroswagginz.com/skills](https://bilbroswagginz.com/skills).

## Install

```bash
git clone https://github.com/SubliminalCoding/bilbroswagginz-skills.git
cp -r bilbroswagginz-skills/skills/* ~/.claude/skills/
```

Start a new Claude Code session. The skills show up in the available-skills list, and Claude picks them from their `description` frontmatter. You can also call one directly, for example `/forge` or `/bug-zero`.

Want only a few? Copy just those folders. The `qwen-*` skills share rules kept in `qwen-forge`, so install `qwen-forge` alongside any of them.

Dependencies: [beads](DEPENDENCIES.md) (`br` + `bv`) is the one hard requirement for the skills that plan and track work. Everything else is optional, and each skill says what it falls back to. See [DEPENDENCIES.md](DEPENDENCIES.md).

## Philosophy

Four principles run through the pack. A skill that expresses none of them does not belong here.

1. **Deposit before work.** Client deals do not start on handshakes. Releases do not tag without a named recipient. Features do not get polished without a user.
2. **Fix the bug, not the category.** Every diff is as small as the problem. No drive-by refactors, no speculative abstractions, no "while you're here" cleanup.
3. **No slop.** No AI-tell phrasing, no em-dash decoration, no hedge questions, no fake enthusiasm. Direct imperatives and concrete rules. If a rule has a reason, state it next to the rule.
4. **The recipient owns the acceptance test.** Whoever the work is for must be able to run the test themselves. If only the author can tell whether it worked, it hasn't worked.

## Skills

### Plan & ship

| Skill | What it does |
|-------|--------------|
| [`/advisor-council`](skills/advisor-council/SKILL.md) | Have several advisor personas argue a hard call, then get one synthesized recommendation. |
| [`/ai-receptionist-tenant-onboarding`](skills/ai-receptionist-tenant-onboarding/SKILL.md) | Add a new business to a multi-tenant AI phone agent and prove it is ready for real calls. |
| [`/beadsmith`](skills/beadsmith/SKILL.md) | Turn an approved plan into trackable work. |
| [`/build-in-public-scope-guard`](skills/build-in-public-scope-guard/SKILL.md) | Sort viewer suggestions during a build stream and decline the distracting ones politely. |
| [`/consulting-sales-playbook`](skills/consulting-sales-playbook/SKILL.md) | Write proposals, follow up and negotiate scope without drifting into unpaid or equity-only work. |
| [`/deposit-first-shipping`](skills/deposit-first-shipping/SKILL.md) | Keep product work tied to real customer demand. |
| [`/dueling-idea-forge`](skills/dueling-idea-forge/SKILL.md) | Challenge ideas across multiple AI models. |
| [`/forge`](skills/forge/SKILL.md) | Take a project from idea to release. |
| [`/idea-forge`](skills/idea-forge/SKILL.md) | Turn project context into a focused improvement backlog. |
| [`/kickstart`](skills/kickstart/SKILL.md) | Turn a new idea into a working project foundation. |
| [`/problem-breakthrough`](skills/problem-breakthrough/SKILL.md) | Find a better path through a stubborn blocker. |
| [`/project-autopilot`](skills/project-autopilot/SKILL.md) | Keep a project moving toward its intended outcome. |
| [`/qwen-ship-it`](skills/qwen-ship-it/SKILL.md) | Do the reversible release prep locally and list every publish step for you to approve. |
| [`/reality-check`](skills/reality-check/SKILL.md) | Compare the project vision with what actually works. |
| [`/ship-it`](skills/ship-it/SKILL.md) | Prepare a working project for release. |

### Code quality

| Skill | What it does |
|-------|--------------|
| [`/agent-exploit-paths`](skills/agent-exploit-paths/SKILL.md) | Find dangerous paths from untrusted input to privileged actions. |
| [`/audit-lens`](skills/audit-lens/SKILL.md) | Review a codebase through a focused quality lens. |
| [`/bug-sweep`](skills/bug-sweep/SKILL.md) | Find and fix bugs across a chosen code surface. |
| [`/bug-zero`](skills/bug-zero/SKILL.md) | Run a thorough project-wide bug hunt. |
| [`/incident-zero`](skills/incident-zero/SKILL.md) | Stabilize production incidents and prevent repeats. |
| [`/migrate-proof`](skills/migrate-proof/SKILL.md) | Move systems safely while protecting existing behavior. |
| [`/no-slop-bug-hunter`](skills/no-slop-bug-hunter/SKILL.md) | Fix the actual bug without unrelated changes. |
| [`/qwen-bug-zero`](skills/qwen-bug-zero/SKILL.md) | Find, confirm and fix bugs one at a time with regression tests, on a local model. |
| [`/security-sweep`](skills/security-sweep/SKILL.md) | Find and fix high-impact SaaS security risks. |
| [`/simplify-proof`](skills/simplify-proof/SKILL.md) | Reduce code complexity without changing behavior. |
| [`/test-max`](skills/test-max/SKILL.md) | Make a test suite more trustworthy. |
| [`/trace-it`](skills/trace-it/SKILL.md) | Make intermittent failures easier to locate. |

### Performance

| Skill | What it does |
|-------|--------------|
| [`/hotspot-scout`](skills/hotspot-scout/SKILL.md) | Find and rank the biggest performance bottlenecks. |
| [`/optimize-proof`](skills/optimize-proof/SKILL.md) | Turn measured bottlenecks into verified speedups. |
| [`/perf-max`](skills/perf-max/SKILL.md) | Improve performance across the whole project. |
| [`/qwen-perf-max`](skills/qwen-perf-max/SKILL.md) | Profile, optimize the top hotspot and prove behavior is unchanged, on a local model. |

### Design & writing

| Skill | What it does |
|-------|--------------|
| [`/design-intel`](skills/design-intel/SKILL.md) | Set a clear visual direction for a new interface. |
| [`/polish-pass`](skills/polish-pass/SKILL.md) | Make an existing interface feel more refined. |
| [`/qwen-ux-maximizer`](skills/qwen-ux-maximizer/SKILL.md) | Fix concrete, checkable UI issues locally and leave taste decisions to you. |
| [`/readme-real`](skills/readme-real/SKILL.md) | Write accurate READMEs that sound human. |
| [`/stream-demo-prep`](skills/stream-demo-prep/SKILL.md) | Rehearse a demo, clean up your screen and have a fallback ready before going live. |
| [`/unslop`](skills/unslop/SKILL.md) | Remove AI-sounding patterns while preserving meaning. |
| [`/ux-maximizer`](skills/ux-maximizer/SKILL.md) | Raise a product's overall UI and UX quality. |
| [`/ux-scan`](skills/ux-scan/SKILL.md) | Find and prioritize usability and accessibility issues. |

### Agents & automation

| Skill | What it does |
|-------|--------------|
| [`/advisor-persona-build`](skills/advisor-persona-build/SKILL.md) | Turn a real person's documented method into a focused decision-making skill. |
| [`/agentic-loops`](skills/agentic-loops/SKILL.md) | Run repeatable agent workflows on a cadence or goal. |
| [`/deepen`](skills/deepen/SKILL.md) | Apply a skill repeatedly for a deeper result. |
| [`/local-llm-ops`](skills/local-llm-ops/SKILL.md) | Operate local AI models with fewer surprises. |
| [`/memguard`](skills/memguard/SKILL.md) | Prevent heavy AI workloads from freezing the machine. |
| [`/memory-forge`](skills/memory-forge/SKILL.md) | Build a reliable memory system for AI agents. |
| [`/qwen-forge`](skills/qwen-forge/SKILL.md) | Detect a project's stage and run the matching local-model skill, with no API tokens. |
| [`/qwen-kickstart`](skills/qwen-kickstart/SKILL.md) | Scaffold a new project with a small local model: intent, simplest stack, working skeleton. |
| [`/qwen-project-autopilot`](skills/qwen-project-autopilot/SKILL.md) | Work through ready tasks one small verified change at a time on a local model. |
| [`/recall`](skills/recall/SKILL.md) | Bring the right past context into the current session. |
| [`/session-mine`](skills/session-mine/SKILL.md) | Find useful decisions and patterns in past agent sessions. |
| [`/skillify`](skills/skillify/SKILL.md) | Turn a solved problem into a reusable skill. |
| [`/swarm-run`](skills/swarm-run/SKILL.md) | Coordinate multiple coding agents on one project. |

## Contributing

Issues and pull requests are welcome. Keep changes in the pack's voice: a rule, the reason next to it, no filler. A new skill needs a `SKILL.md` with `name` and a `description` that says when to use it.

## License

MIT. See [LICENSE](LICENSE). The format takes inspiration from Jeffrey Emanuel's `jsm` skill pack; the content is original.
