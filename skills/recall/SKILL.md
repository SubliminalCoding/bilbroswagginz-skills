---
name: recall
description: >-
  Load relevant memories for the current project/context. Auto-detects project from
  working directory, or accepts a keyword query. Use when starting a session, switching
  projects, or needing context on a person, project, or decision. Works from any
  terminal.
trigger: when starting work in a project directory, when the user asks "what do we know about X", when context from prior sessions would help
---

# Recall: Project-Aware Memory Loader

Load relevant memories based on current working directory, conversation topic, or keyword search. This is the READ half of the memory loop: `memory-forge` curates the store; recall loads the right slice of it at the right moment. Feed it a store built by `memory-forge` (sourced via `session-mine`) and every session starts already oriented.

## Usage

- `/recall`: auto-detect project from CWD, load matching memories
- `/recall <query>`: search all memories by keyword (e.g., `/recall <person>`, `/recall <project>`)

## Instructions

### Step 1: Determine context

Run `pwd` to get the current working directory. Also check if ARGUMENTS were provided (keyword query).

### Step 2: Load the memory index

Read the memory index at your memory root, e.g.:
```
~/.claude/projects/<home-slug>/memory/MEMORY.md
```

### Step 3: Match memories to load

**If CWD matches a known project directory (check substring match against the path):**

Use the mappings below. Load ALL matched memory files plus the always-load set.

**If a keyword query was provided in ARGUMENTS:**

Search (grep) across all `.md` files in the memory directory for the keyword. Load any files that match.

**If neither CWD nor query produces matches:**

Load the always-load set and present the full memory index so the user can pick what's relevant.

### Step 4: Read and present

Read each matched memory file from the memory directory.

Present a concise context briefing. Format:

```
## Recall: [project name or query]

**Project context:**
- [key facts, current state, decisions made]

**User preferences / feedback:**
- [relevant feedback memories that apply]

**Warnings / red flags:**
- [if any: behavioral patterns, known risks, known issues]

**Key files:**
- [relevant file paths for this project]
```

Keep it tight. The point is to get oriented fast, not to read every word of every memory back.

---

## Always-Load Set

These memories are relevant in every context. Always read them:

| File | Why |
|------|-----|
| `user_profile.md` | Who the user is, how to collaborate with them |
| `project_machine_setup.md` | Machine context, services, what's available |

Also load ALL `feedback_*.md` files; they contain behavioral guidance that applies across projects.

---

## Project Directory Mappings

Match the current working directory (substring match) against these patterns. A directory can match multiple entries. Maintain this table per machine; the entries below are the shape, not the content:

| Directory contains | Memory files to load | Notes |
|--------------------|---------------------|-------|
| `<client-project>` | `project_<client>.md` | Engagement context, deal structure, open threads |
| `<agent-name>` | `project_<agent>.md`, `reference_vault.md` | The agent + the vault it writes to |
| `<app-name>` | `project_<app>.md`, `feedback_<constraint>.md` | App context + any standing constraint that applies |

### Home directory fallback

If CWD is just `~` with no project subfolder, load the full memory index and ask what the user is working on.

---

## Keyword Search

When ARGUMENTS contain a query string:

1. Use Grep to search all `.md` files in the memory directory for the keyword
2. Read any files that match
3. Also check your knowledge base (e.g. a notes-vault search tool) if the keyword doesn't match any memory files
4. Present combined results

---

## Maintaining This Skill

When new memory files are created or projects change:

1. Update the **Project Directory Mappings** table above
2. Update the **Always-Load Set** if new feedback memories are broadly applicable
3. The keyword search is self-maintaining (greps across all memory files)

---

## Example Output

```
## Recall: client-poc

**Project context:**
- Consulting engagement with <client>
- Phase 1: paid pilot; later phases scoped separately
- Demo held <date>, follow-up emails sent

**User preferences:**
- Only 1 local model loaded at a time on this box
- All local models, no cloud API in production demos

**Warnings:**
- Get every scope change in writing

**Key files:**
- ~/client-poc/: webapp, proposals, emails, test data
- ~/client-poc/FOLLOWUP-EMAIL.md: sent emails
```
