---
name: fewer-permission-prompts
description: Scan Claude Code transcripts for common read-only Bash and MCP tool calls, then add a prioritized allowlist to the project .claude/settings.json to reduce permission prompts without granting broad access.
origin: claude-code-harness
tools: Read, Write, Edit, Bash, Glob
---

# Fewer Permission Prompts

Reduce Claude Code permission prompts by scanning transcripts for safe, repeated tool calls and adding a targeted allowlist to `.claude/settings.json`.

## When to Use

- User is tired of approving the same tool calls every session
- User asks to "reduce prompts" or "auto-allow common commands"
- You notice you're repeatedly requesting permission for read-only operations
- Setting up a new project where common safe commands are known in advance

**Do not use to:**
- Grant broad write access (`Bash(**)`) — keep allowlists narrow
- Bypass prompts for destructive operations (deletions, force-pushes, drops)
- Skip prompts for commands that touch shared infrastructure

## How It Works

### 1. Scan Transcripts for Repeated Tool Calls

Look at recent session transcripts (if available) to identify tool calls that:
- Are read-only (no side effects)
- Appear repeatedly across sessions
- Are consistently approved by the user

Common candidates:
```
Bash(git status)
Bash(git log *)
Bash(git diff *)
Bash(ls *)
Bash(find . *)
Bash(grep *)
Bash(node --version)
Bash(npm list *)
Read(**)
Glob(**)
```

### 2. Prioritize by Safety

| Safety Level | Examples | Action |
|-------------|----------|--------|
| Safe — read-only | `git status`, `ls`, `grep`, `Read` | Allow |
| Safe — idempotent | `git diff`, `git log`, `node -e` | Allow with pattern |
| Potentially destructive | `rm`, `git reset`, `git push` | Do NOT allow |
| Always destructive | `git push --force`, `DROP TABLE` | Never allow |

### 3. Add to Project Settings

Edit `.claude/settings.json` (project-scoped, checked into git) or `.claude/settings.local.json` (personal, gitignored):

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git log *)",
      "Bash(git diff *)",
      "Bash(git diff --stat *)",
      "Bash(ls *)",
      "Bash(ls -la *)",
      "Bash(find . *)",
      "Bash(grep *)",
      "Bash(node --version)",
      "Bash(npm list *)",
      "Bash(npx tsc --noEmit)",
      "Read(**)",
      "Glob(**)"
    ]
  }
}
```

### 4. Validate Scope

Before committing, confirm each entry in the allowlist:
- Uses the narrowest pattern that covers the actual use case
- Does not accidentally allow destructive variations
- Is appropriate to commit to the repo (use `settings.local.json` for personal preferences)

## Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `Bash(git status)` | Exact command only |
| `Bash(git log *)` | `git log` with any arguments |
| `Bash(npm run *)` | Any npm script |
| `Read(**)` | Any file read |
| `Bash(find . -name *)` | `find` with name filter |

## Settings File Choice

| File | Use When |
|------|----------|
| `.claude/settings.json` | Team-wide allowlist — safe for everyone on the project |
| `.claude/settings.local.json` | Personal allowlist — your workflow, not teammates' |
| `~/.claude/settings.json` | Global — applies across all your projects |

## Related Skills

- `update-config` — Broader settings.json configuration (hooks, env vars)
- `configure-ecc` — ECC-specific hook profile configuration
