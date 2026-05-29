---
name: verify
description: Verify that a code change actually works by running the app and observing behavior. Use when asked to verify a PR, confirm a fix works, test a change manually, check that a feature works, or validate local changes before pushing.
origin: claude-code-harness
tools: Read, Bash, Glob
---

# Verify

Verify that a code change does what it's supposed to do by running the app and observing actual behavior — not just reading code or running type checks.

## When to Use

- User asks to "verify" a PR or change
- User wants to confirm a fix actually works end-to-end
- User wants manual testing of a feature before pushing
- User wants to validate local changes before a code review
- Type checking and tests pass but you need to confirm the feature works in practice

**Not a substitute for:**
- Automated test suites — run those separately
- Code review — use `code-review` for correctness analysis
- Security review — use `security-review` for vulnerability scanning

## Verification Workflow

### 1. Understand What Changed

```bash
git diff --stat HEAD~1
git diff HEAD~1
```

Identify the feature, fix, or behavior change to verify.

### 2. Determine the Golden Path

For a given change, identify:
- **Happy path**: the primary use case that should work
- **Edge cases**: boundary conditions or error states affected by the change
- **Regression surface**: existing features that could have broken

### 3. Run the App

Start the application in a mode appropriate for the change:

```bash
# Web server
npm run dev

# CLI tool
node scripts/ecc.js <command>

# Test suite (as a smoke check)
npm test
```

### 4. Exercise the Change

Interact with the app to trigger the changed code path. Observe:
- Does the output match expectations?
- Are there unexpected errors in logs or console?
- Does the UI render correctly (if applicable)?

### 5. Check for Regressions

Exercise adjacent features that could have been affected. Look for:
- Unintended side effects
- Broken imports or missing exports
- Changed behavior in unmodified code paths

### 6. Report Findings

After verification:
- State clearly whether the change works as intended
- List any edge cases tested and their outcomes
- Call out any regressions found
- If you cannot test the UI (no display), say so explicitly rather than claiming success

## Verification by Change Type

| Change Type | Key Things to Verify |
|-------------|---------------------|
| Bug fix | Reproduce the original bug, then confirm it's gone |
| New feature | Exercise the golden path + at least one edge case |
| Refactor | All existing behavior preserved; no output changes |
| Config change | App starts cleanly; relevant feature works |
| Dependency update | No runtime errors; critical paths still work |

## Important Notes

- **Type checking is not verification** — `tsc --noEmit` passing means the types are consistent, not that the feature works.
- **Tests are not verification** — unit tests passing means individual units work in isolation, not that the integrated app works.
- If you cannot run the app (missing deps, CI-only environment), say so rather than skipping verification.

## Related Skills

- `code-review` — Correctness analysis of the diff
- `run` — Launch and drive the app for broader exploration
- `security-review` — Security audit of pending changes
