---
name: keybindings-help
description: Customize Claude Code keyboard shortcuts by editing ~/.claude/keybindings.json. Use when the user wants to rebind keys, add chord shortcuts, change the submit key, or otherwise modify keybindings.
origin: claude-code-harness
tools: Read, Write, Edit
---

# Keybindings Help

Customize Claude Code keyboard shortcuts by editing `~/.claude/keybindings.json`. Supports single key bindings, chord (multi-key) sequences, and context-sensitive bindings.

## When to Use

- User wants to rebind a key ("rebind ctrl+s to X")
- User wants to add a chord shortcut ("add a chord binding")
- User wants to change the submit key (default: Enter)
- User wants to customize keybindings in general
- User references `~/.claude/keybindings.json`

## Keybindings File Location

```
~/.claude/keybindings.json
```

This is a global file — keybindings apply across all projects.

## File Format

```json
[
  {
    "key": "ctrl+shift+enter",
    "command": "workbench.action.chat.submit",
    "when": "chatInputFocus"
  },
  {
    "key": "ctrl+k ctrl+r",
    "command": "code-review",
    "when": "inChat"
  }
]
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `key` | Yes | Key combination (see modifiers below) |
| `command` | Yes | The command or slash command to invoke |
| `when` | No | Context clause limiting when the binding is active |

### Key Modifiers

| Modifier | Alias | Notes |
|----------|-------|-------|
| `ctrl` | `cmd` (macOS) | Control key |
| `shift` | — | Shift key |
| `alt` | `opt` (macOS) | Alt/Option key |
| `meta` | — | Windows/Super key |

### Chord Bindings

Chords are two-key sequences separated by a space:

```json
{ "key": "ctrl+k ctrl+c", "command": "toggleComment" }
```

The user presses `Ctrl+K`, releases, then presses `Ctrl+C`.

## Common Customizations

### Change Submit Key to Shift+Enter

```json
[
  { "key": "shift+enter", "command": "chat.submit" },
  { "key": "enter", "command": "chat.newline" }
]
```

### Add Quick Code Review Chord

```json
[
  { "key": "ctrl+k ctrl+r", "command": "/code-review" }
]
```

### Disable a Default Binding

Set `command` to `""` (empty string) or `null` to disable:

```json
[
  { "key": "ctrl+enter", "command": "" }
]
```

## Workflow

1. **Read the current file**: `~/.claude/keybindings.json` (may not exist yet)
2. **Identify the binding** to add, change, or remove
3. **Edit or create** the file with the new binding
4. **Restart Claude Code** for changes to take effect

## Related Skills

- `update-config` — Edit `settings.json` for hooks, permissions, and env vars
