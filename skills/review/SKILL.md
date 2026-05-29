---
name: review
description: Review a pull request. Fetches the PR diff, reads changed files in context, and produces ranked findings covering correctness, test coverage, and adherence to project conventions. Can post findings as inline PR comments.
origin: claude-code-harness
tools: Read, Bash, Glob, Grep
---

# Review

Review a pull request end-to-end: fetch the diff, read changed files with full context, and produce ranked findings covering correctness bugs, test coverage, and project convention adherence.

## When to Use

- User asks to "review" a PR (by number or URL)
- User wants a thorough code review before merging
- User wants findings posted as inline GitHub PR comments
- Reviewing a PR from a teammate or dependency update

**Invoke via:** `/review [PR# or URL]`

For a quick diff review without a PR, use `code-review` instead.

## Review Workflow

### 1. Fetch the PR

```bash
# Fetch PR metadata and diff
gh pr view <PR#> --json title,body,author,baseRefName,headRefName,files,commits

# Get the full diff
gh pr diff <PR#>

# Checkout the branch for deeper analysis
gh pr checkout <PR#>
```

### 2. Read the PR Description

Understand:
- What problem does this PR solve?
- What approach did the author take?
- Are there any noted trade-offs or areas for feedback?
- Is there a linked issue or spec?

A PR description that doesn't explain the "why" is itself a finding.

### 3. Analyze Changed Files

For each file in the diff:
1. Read the full file (not just the diff) for context
2. Check how the change fits into the surrounding code
3. Identify callsites of changed functions
4. Check tests for the changed behavior

### 4. Correctness Analysis

For each change, verify:
- **Logic** — does the implementation match the intent?
- **Edge cases** — what happens at boundaries (empty, null, max)?
- **Error handling** — are failures handled at system boundaries?
- **Concurrency** — any race conditions or shared state issues?
- **API contracts** — are external/internal APIs used correctly?

### 5. Test Coverage

- Are the changed behaviors covered by tests?
- Do existing tests still pass with this change?
- Are edge cases tested?
- Is the happy path clearly exercised?

### 6. Convention Adherence

Check against project conventions (from CLAUDE.md and existing patterns):
- File naming and structure
- Import style
- Error handling patterns
- Logging conventions
- Documentation standards

### 7. Rank and Report Findings

Order findings by:
1. **Severity** — bugs that break production first
2. **Confidence** — high-confidence findings before speculative
3. **Scope** — findings affecting many users/callsites before isolated ones

For each finding:
- **Location**: `path/to/file.ts:42`
- **Severity**: critical / high / medium / low
- **Issue**: what is wrong and why it matters
- **Suggestion**: concrete fix, not vague advice

## Posting Inline Comments

To post findings as inline PR comments:

```bash
# Post a general comment
gh pr comment <PR#> --body "Overall findings: ..."

# Post an inline comment at a specific line
gh api repos/:owner/:repo/pulls/:pr/comments \
  --method POST \
  --field body="Issue: ..." \
  --field commit_id="<sha>" \
  --field path="src/file.ts" \
  --field line=42
```

## What NOT to Review

- Style issues the linter should catch automatically
- Naming preferences with no correctness impact
- Changes outside the PR scope (mention as a separate suggestion, don't block)
- Architectural re-designs beyond the PR's stated scope

## Review Summary Format

```
## Summary

[1-2 sentence overall assessment]

## Findings

### Critical
- `src/auth.ts:87` — [issue] [suggestion]

### High
- `src/api/users.ts:23` — [issue] [suggestion]

### Medium
- `src/utils/format.ts:11` — [issue] [suggestion]

## Positive Notes

[Things done especially well — optional but valuable]
```

## Related Skills

- `code-review` — Review the current working tree diff (no PR required)
- `security-review` — Security-focused review of pending changes
- `verify` — Confirm the change works by running the app
