---
name: no-slop-bug-hunter
description: >-
  Multi-pass debugging that fixes THE bug, not the category: no drive-by refactors,
  no defensive try/except padding, no feature-flag shortcuts, no abstractions added
  "while you're here." Use when hunting a reproducible bug, triaging a flaky test,
  investigating a production error, or reviewing a proposed fix for a specific
  defect.
---

# No-Slop Bug Hunter

Multi-pass bug hunting without the slop. Most debugging agents do two things wrong: they fail to reproduce before fixing, and they ship surrounding "improvements" that expand the diff beyond the bug. This skill forbids both.

## The contract

Before any code change, the following must be true:

1. **The bug is reproduced.** A failing test, a one-line shell command, or a written sequence of clicks that reliably produces the symptom. If it can't be reproduced, it hasn't been found. Stop and keep investigating.
2. **The root cause is named.** One sentence. "The cache key omits the tenant id, so tenant A reads tenant B's response." Not "error handling is insufficient." If the cause can't be stated concretely, the fix is a guess.
3. **The scope of the fix is named.** The exact files and functions that will change. Anything outside that list requires a separate commit or a separate skill invocation.

If any of those three are missing, the agent is not debugging; it's sloshing around in the code.

## Hard rules while fixing

- **Fix THE bug. Not the category.** If a `null` check is missing on line 42, add it on line 42. Do not audit the file for other missing null checks in the same commit. A separate audit is a separate task.
- **Do not add defensive try/except blocks that swallow the symptom.** Catching the exception that revealed the bug is hiding the bug, not fixing it. Catch only where the recovery behavior is specified and correct.
- **Do not add feature flags as fixes.** A feature flag that gates the broken code path is not a fix; it's a postponement. If the behavior is wrong, change the behavior.
- **Do not refactor surrounding code.** Renaming variables, extracting helpers, "improving" the nearby structure: none of that belongs in a bug fix. It expands the review surface, obscures what actually changed, and risks introducing new bugs.
- **Do not add comments explaining the bug.** `// fix for issue #123` rots. `// this handles the case where X` explains what the code already shows. If the fix is non-obvious, the commit message is where that context lives, not a comment.
- **Do not add logging as the fix.** Logging is how you found the bug. It is not the bug's fix.
- **The commit message names what was broken and why the fix works.** Not "improve robustness." Not "handle edge case." Name the specific broken invariant and the specific change that restores it.

## The three-pass structure

Debugging is iterative. Cap it.

**Pass 1: Reproduce and isolate.**
- Get the bug to trigger on demand.
- Bisect if needed (git bisect, binary-search logs, comment out halves of the flow).
- End state: a minimal trigger and a named suspect.

**Pass 2: Verify the cause.**
- Prove the named suspect is the cause by changing it and confirming the bug goes away.
- Prove no other change is needed by reverting your change and confirming the bug returns.
- End state: a one-line diff that makes the failing repro pass.

**Pass 3: Minimize and land.**
- Clean the diff to the smallest correct form.
- Add a test that pins the fix (if the codebase has a test suite; don't add one just for this).
- Write the commit message. Name the broken invariant. Name the fix.

**If three passes don't solve it, stop and escalate.** The next move is not more passes; it's a different approach: rubber-duck with the operator, read the upstream library source, check if the bug is actually in a dependency, or accept that the reproduction isn't as reliable as you thought and go back to Pass 1.

## Anti-patterns that look like progress

These are the moves that feel productive but waste the session:

- **The "grep and patch" sweep.** Searching for all instances of a similar pattern and patching them preemptively. You have one repro. Fix the one bug. Other instances are a separate audit.
- **The "while I'm here" cleanup.** Reformatting, reordering imports, renaming a misspelled variable. Not in this commit.
- **The "just in case" guard.** Adding validation for inputs that can't occur under the current call graph. Every guard you add is code that will be read and maintained forever. Only validate at system boundaries.
- **The "let's make it more robust" rewrite.** A bug fix is not a license to rewrite the module. If the module needs a rewrite, that is a different conversation with different scope.

When any of these urges appear during a fix, name them and resist them. Log them as follow-up tasks if they're real, and ship the minimal fix now.

## What a good fix looks like

```
- if user.role == "admin":
+ if user.role == "admin" and user.tenant_id == resource.tenant_id:
```

One line. Repro before: admin from tenant A can read tenant B's resource. Repro after: 403. Commit message names the broken invariant (role check omitted tenant scoping) and the fix (added the tenant equality check). Nothing else in the diff.

That is the target shape for every fix this skill produces.
