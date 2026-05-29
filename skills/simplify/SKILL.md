---
name: simplify
description: Review the current diff and apply the fixes — equivalent to /code-review --fix. Finds correctness bugs, unnecessary complexity, and reuse opportunities, then applies the changes directly to the working tree.
origin: claude-code-harness
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Simplify

Review the current branch diff and apply fixes directly to the working tree. Equivalent to `/code-review --fix`. Targets correctness bugs, unnecessary complexity, and missed reuse opportunities.

## When to Use

- User asks to "simplify" the current changes
- User wants code review findings applied automatically
- User says "clean up the diff" or "fix the obvious issues"
- After a first-pass implementation that needs tidying before final review

**Invoke via:** `/simplify`

This is a shortcut for `/code-review --fix`. For review-only (no edits), use `code-review`.

## What Simplify Does

### 1. Review the Diff

Reads the current diff against the base branch:

```bash
git diff main...HEAD
```

Analyzes for:
- **Correctness bugs** — logic errors, null dereferences, wrong API usage
- **Unnecessary complexity** — verbose code with simpler equivalents
- **Missed reuse** — duplicated logic that an existing utility already handles
- **Dead code** — imports, variables, or branches that can't be reached

### 2. Apply Fixes

For each finding, directly edits the affected file using minimal, targeted changes. Does not:
- Reformat code the linter should handle
- Add abstractions beyond what the task requires
- Refactor code outside the diff scope
- Add comments explaining what the code does

### 3. Report What Changed

After applying fixes, reports:
- Which files were edited
- What was changed and why
- Any findings that were too uncertain to auto-apply (flagged for human review)

## Simplification Principles

These guide what gets fixed and what gets left alone:

- **Three similar lines beat a premature abstraction** — don't DRY three callsites into a helper unless it's clearly the right move
- **Don't design for hypothetical future requirements** — only simplify what's actually in the diff
- **No half-finished implementations** — if a simplification requires follow-up work, flag it instead of applying it
- **Trust the framework** — don't add error handling for scenarios the framework already handles
- **No backwards-compatibility shims** — if something is unused, delete it cleanly

## Scope Boundaries

Simplify operates only on **code in the current diff**. It does not:
- Modify files not touched by the current branch
- Change test files unless the test is directly related to a fixed bug
- Touch configuration files unless misconfiguration is the bug

## Related Skills

- `code-review` — Review without applying fixes (for human sign-off first)
- `verify` — Confirm the simplified code still works
- `security-review` — Security pass after simplification
