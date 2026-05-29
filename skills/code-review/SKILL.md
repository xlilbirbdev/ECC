---
name: code-review
description: Review the current diff for correctness bugs and reuse/simplification/efficiency cleanups at a given effort level. Pass --comment to post findings as inline PR comments, or --fix to apply findings to the working tree. Supports low/medium/high/max/ultra effort levels.
origin: claude-code-harness
tools: Read, Bash, Glob, Grep
---

# Code Review

Review the current branch diff for correctness bugs, reuse opportunities, simplification wins, and efficiency improvements. Produces ranked findings with optional inline PR comments or automatic fixes.

## When to Use

- User asks for a code review of the current branch or a specific PR
- User wants findings posted as inline PR comments (`--comment`)
- User wants findings automatically applied to the working tree (`--fix`)
- User specifies an effort level: low, medium, high, max, or ultra

**Invoke via:** `/code-review [effort] [--comment] [--fix] [PR#]`

**Ultra mode** (`/code-review ultra` or `/code-review ultra <PR#>`): launches a multi-agent cloud review. Billed and user-triggered only — do not invoke via Bash.

## Effort Levels

| Level | Coverage | Best For |
|-------|----------|----------|
| `low` | High-confidence, obvious bugs only | Quick sanity check before merge |
| `medium` | Moderate coverage, clear wins | Standard PR review |
| `high` | Broader coverage, may include uncertain findings | Important features or risky changes |
| `max` | Maximum breadth, includes speculative findings | Critical systems, security-sensitive changes |
| `ultra` | Deep multi-agent cloud review | Large PRs, pre-release audits |

Default effort: `medium`.

## Review Workflow

### 1. Identify the Diff

```bash
# Current branch vs main
git diff main...HEAD

# Specific PR (with gh CLI)
gh pr diff <PR#>

# Staged changes only
git diff --cached
```

### 2. Collect Context

Before reviewing, read:
- Files changed in the diff
- Adjacent files that depend on changed code
- Existing tests for changed functionality
- Any CLAUDE.md conventions that apply

### 3. Apply Effort-Appropriate Analysis

**Correctness bugs (all levels):**
- Logic errors, off-by-one errors, null/undefined dereferences
- Incorrect API usage, wrong argument order
- Race conditions, improper async/await
- Missing error handling at system boundaries

**Simplification/reuse (medium+):**
- Duplicated logic that could use an existing utility
- Overly complex implementations with simpler alternatives
- Dead code or unused imports

**Efficiency (high+):**
- N+1 queries, unnecessary re-renders, wasteful allocations
- Missing memoization or caching opportunities

**Speculative (max only):**
- Hypothetical future issues
- Style suggestions beyond project conventions

### 4. Rank Findings

Order findings by:
1. **Severity** — bugs that would break production first
2. **Confidence** — high-confidence findings before speculative ones
3. **Impact** — changes that affect many callsites before isolated issues

### 5. Output Findings

For each finding:
- **File and line reference** — `path/to/file.ts:42`
- **Issue description** — what is wrong and why
- **Suggested fix** — concrete code change, not vague advice

## Flags

### `--comment` — Post as Inline PR Comments

Posts each finding as an inline comment on the GitHub PR at the relevant line. Requires `gh` CLI authenticated. Does not modify the working tree.

```bash
gh pr review <PR#> --comment --body "..."
# or per-line:
gh api repos/:owner/:repo/pulls/:pr/comments ...
```

### `--fix` — Apply Findings

After generating findings, applies fixes directly to the working tree. Equivalent to running `/simplify` after the review. Use only when findings are high-confidence.

## What NOT to Review

- Formatting — let the linter handle it
- Naming preferences with no correctness impact
- Architectural changes outside the PR scope
- Hypothetical future requirements (at low/medium effort)

## Related Skills

- `simplify` — Apply fixes from the current diff review
- `verify` — Confirm a change works by running the app
- `security-review` — Security-focused review of pending changes
- `agent-architecture-audit` — Deep audit for agent/LLM applications
