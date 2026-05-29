---
name: loop
description: Run a prompt or slash command on a recurring interval using /loop. Omit the interval to let the model self-pace. Use for recurring tasks, status polling, or repeating a command at set intervals (e.g. /loop 5m /foo). Not for one-off tasks.
origin: claude-code-harness
tools: Bash
---

# Loop

Run a prompt or slash command on a recurring interval. The harness re-invokes Claude on each tick with the same prompt, maintaining context across iterations.

## When to Use

- User wants to set up a recurring task ("check the deploy every 5 minutes")
- User wants to poll for status ("keep watching until the build passes")
- User wants to run something repeatedly on a schedule ("run /babysit-prs every 10 minutes")
- User wants Claude to self-pace iterations of a task (omit the interval)

**Do not use for:**
- One-off tasks — just run the task directly
- Tasks that should run on a cron schedule after the session ends — use `schedule` instead

## Invocation

```
/loop [interval] [prompt or /skill]
```

### Examples

```
/loop 5m /code-review          # Run /code-review every 5 minutes
/loop 1m check deploy status   # Poll deploy status every minute
/loop /babysit-prs             # Self-paced PR monitoring
/loop 30s npm test             # Run tests every 30 seconds
```

### Interval Format

| Format | Meaning |
|--------|---------|
| `30s` | Every 30 seconds |
| `5m` | Every 5 minutes |
| `1h` | Every hour |
| (omitted) | Self-paced — model decides the interval |

## Self-Paced Loops

When no interval is given, Claude chooses its own delay between iterations based on what it's waiting for. Self-pacing is useful when:
- The work has variable duration (some iterations are fast, some slow)
- You're waiting for an external event with an unknown ETA
- The natural cadence depends on what Claude finds in each iteration

The model picks delays to balance responsiveness against cost:
- Warm cache window (< 5 min): check frequently
- Long waits (> 5 min): space out to avoid cache misses

## Stopping a Loop

- Press `Ctrl+C` or close the session to stop the loop
- The loop exits automatically when the task is complete (model omits the next wake-up call)
- Use a condition in your prompt: "keep running until the CI is green"

## Loop vs Schedule

| Feature | `/loop` | `schedule` |
|---------|---------|-----------|
| Runs while session is active | Yes | No |
| Runs after session ends | No | Yes |
| Requires Claude Code open | Yes | No |
| Good for | Live monitoring, active polling | Nightly jobs, recurring reports |

## Patterns

### Poll Until Done

```
/loop 1m check if the deploy at staging.example.com is healthy; stop when it is
```

### Recurring Review

```
/loop 10m /code-review low
```

### Self-Paced Build Watch

```
/loop watch the CI output and report when the build passes or fails
```

## Related Skills

- `schedule` — Recurring remote agents that run after the session ends
- `autonomous-loops` — Patterns for fully autonomous agent loops
- `update-config` — Configure hooks for event-driven automation (instead of polling)
