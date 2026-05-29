---
name: update-config
description: Configure Claude Code harness via settings.json. Use for automated behaviors (hooks), permissions, env vars, hook troubleshooting, and any changes to settings.json or settings.local.json. For simple settings like theme/model, suggest /config instead.
origin: claude-code-harness
tools: Read, Write, Edit, Bash
---

# Update Config

Configure the Claude Code harness by editing `settings.json` or `settings.local.json`. This skill covers automated behaviors (hooks), permission allowlists, environment variables, and hook troubleshooting.

## When to Use

- User says "from now on when X", "each time X", "whenever X", "before/after X" — these require hooks, not memory
- User asks to allow a tool or command ("allow npm commands", "add bq permission")
- User wants to move a permission to global or project settings
- User wants to set an environment variable ("set DEBUG=true")
- User is troubleshooting a hook that isn't firing or is blocking
- Any direct edit to `settings.json` or `settings.local.json`

**Do not use for:**
- Simple theme or model changes — suggest `/config` instead
- One-off tasks that don't need persistent automation

## Key Concepts

### Settings File Locations

| File | Scope |
|------|-------|
| `~/.claude/settings.json` | Global — applies to all projects |
| `.claude/settings.json` | Project — checked into the repo |
| `.claude/settings.local.json` | Project-local — gitignored, personal overrides |

### Hooks vs Memory

Automated behaviors ("run X after every edit") **must** be implemented as hooks in `settings.json`. Claude's memory and preferences cannot execute code automatically — the harness does. Memory is for context; hooks are for automation.

### Hook Event Types

| Event | When it fires |
|-------|--------------|
| `PreToolUse` | Before any tool call — can block execution |
| `PostToolUse` | After any tool call completes |
| `Stop` | When Claude finishes a turn |
| `Notification` | On background notifications |

## How It Works

### Adding a Hook

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "tool_name": "Edit" },
        "hooks": [
          { "type": "command", "command": "npx eslint --fix $CLAUDE_TOOL_INPUT_FILE_PATH" }
        ]
      }
    ]
  }
}
```

### Adding a Permission

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git *)",
      "Read(**)"
    ]
  }
}
```

### Setting an Environment Variable

```json
{
  "env": {
    "DEBUG": "true",
    "NODE_ENV": "development"
  }
}
```

## Workflow

1. **Identify the target file** — global (`~/.claude/settings.json`) or project (`.claude/settings.json`)
2. **Read the current file** before editing
3. **Apply the change** — hook, permission, or env var
4. **Validate JSON** — malformed settings silently fail
5. **Test the hook** — trigger the event and confirm it fires

## Hook Troubleshooting

Common reasons a hook doesn't fire:

- **Wrong event type** — `PreToolUse` vs `PostToolUse` vs `Stop`
- **Matcher too narrow** — check `tool_name` spelling (case-sensitive)
- **JSON parse error** — the entire settings file is silently skipped
- **Command exits non-zero** — for `PreToolUse`, this blocks the tool; for others it's logged
- **ECC hook gating** — if using ECC hooks, check `ECC_DISABLED_HOOKS` env var

## Related Skills

- `configure-ecc` — ECC-specific configuration and install profiles
- `autonomous-loops` — Setting up autonomous agent loops via hooks
- `keybindings-help` — Customize Claude Code keyboard shortcuts
