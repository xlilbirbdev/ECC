---
name: schedule
description: Create, update, list, or run scheduled remote agents (routines) that execute on a cron schedule. Use when the user wants to schedule a recurring remote agent, set up automated tasks, or create a one-time scheduled run.
origin: claude-code-harness
tools: Bash
---

# Schedule

Create, update, list, and run scheduled remote agents — called "routines" — that execute on a cron schedule, even when Claude Code is not running.

## When to Use

- User wants to schedule a recurring remote agent ("run this nightly", "every Monday morning")
- User wants to set up an automated task ("generate a weekly report")
- User wants a one-time scheduled run ("remind me to check X tomorrow", "run this once at 3pm")
- User wants to list or manage existing scheduled routines

**Do not use for:**
- Live session polling — use `loop` instead
- One-off tasks that should run immediately — just run them now

## Invocation

```
/schedule [create|list|run|update|delete] [options]
```

### Create a Routine

```
/schedule create --name "nightly-review" --cron "0 2 * * *" --prompt "/code-review"
/schedule create --name "weekly-report" --cron "0 9 * * MON" --prompt "generate weekly status report"
/schedule create --at "2024-01-15 14:00" --prompt "check if the migration is complete"
```

### List Routines

```
/schedule list
```

### Run a Routine Now

```
/schedule run nightly-review
```

### Update a Routine

```
/schedule update nightly-review --cron "0 3 * * *"
```

### Delete a Routine

```
/schedule delete nightly-review
```

## Cron Schedule Format

```
┌─── minute (0-59)
│  ┌── hour (0-23)
│  │  ┌─ day of month (1-31)
│  │  │  ┌ month (1-12)
│  │  │  │  ┌ day of week (0-6, Sun=0)
*  *  *  *  *
```

### Common Schedules

| Cron | Meaning |
|------|---------|
| `0 * * * *` | Every hour |
| `0 9 * * *` | Daily at 9am |
| `0 9 * * MON` | Every Monday at 9am |
| `0 2 * * *` | Nightly at 2am |
| `*/15 * * * *` | Every 15 minutes |
| `0 0 1 * *` | First of each month |

## One-Time Scheduled Runs

For a one-time future run, use `--at` instead of `--cron`:

```
/schedule create --at "tomorrow 9am" --prompt "check if the feature flag is ready to enable"
/schedule create --at "2024-12-31 23:59" --prompt "run year-end report"
```

## Schedule vs Loop

| Feature | `schedule` | `loop` |
|---------|-----------|--------|
| Runs after session ends | Yes | No |
| Requires Claude Code open | No | Yes |
| Persistent across reboots | Yes | No |
| Good for | Nightly jobs, weekly reports | Live polling, active monitoring |

## What Routines Can Do

Scheduled routines run a full Claude Code agent with access to:
- All tools available in a normal session
- The project working directory
- Any MCP servers configured for the project

Routines can:
- Run code reviews, generate reports, send notifications
- Check external services and file issues
- Run tests and post results
- Any task you'd give Claude in a normal session

## Related Skills

- `loop` — Run a command on a recurring interval during a live session
- `autonomous-loops` — Patterns for fully autonomous agent loops
- `update-config` — Configure hooks for event-driven automation
