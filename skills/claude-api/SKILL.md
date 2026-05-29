---
name: claude-api
description: Build, debug, and optimize Claude API / Anthropic SDK applications. Covers prompt caching, thinking, tool use, batch processing, streaming, file APIs, citations, and model migrations. Trigger when code imports anthropic or @anthropic-ai/sdk.
origin: claude-code-harness
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Claude API

Build, debug, and optimize applications using the Claude API (Anthropic SDK). All apps built with this skill should include prompt caching.

## When to Activate

**Trigger on:**
- Code that imports `anthropic` or `@anthropic-ai/sdk`
- User asks about the Claude API, Anthropic SDK, or Managed Agents
- User adds or modifies Claude features: caching, thinking, tool use, batch, files, citations, memory
- User asks about a specific model (Opus, Sonnet, Haiku)
- Questions about prompt caching or cache hit rates in an Anthropic SDK project
- Migrating between Claude model versions

**Skip when:**
- Code imports `openai` or another provider SDK
- Filename contains `-openai.py`, `-generic.py`, or similar
- Provider-neutral code with no Anthropic-specific patterns

## Current Models (as of 2025)

| Model | ID | Best For |
|-------|-----|----------|
| Claude Opus 4.7 | `claude-opus-4-7` | Most capable, complex reasoning |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | Balanced capability and speed |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | Fast, cost-efficient |

Default to the latest and most capable model for new applications.

## Prompt Caching (Required)

Every app built with this skill must include prompt caching. Caching reduces cost and latency for repeated system prompts, tool definitions, and long documents.

### Python

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "Your long system prompt here...",
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": "Hello"}]
)
```

### TypeScript

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "claude-opus-4-7",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: "Your long system prompt here...",
      cache_control: { type: "ephemeral" },
    },
  ],
  messages: [{ role: "user", content: "Hello" }],
});
```

### Cache Hit Detection

```python
# Check cache performance
print(response.usage.cache_creation_input_tokens)  # tokens written to cache
print(response.usage.cache_read_input_tokens)       # tokens read from cache
```

## Tool Use

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City name"}
            },
            "required": ["location"]
        }
    }
]

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Paris?"}]
)
```

## Extended Thinking

```python
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000
    },
    messages=[{"role": "user", "content": "Solve this complex problem..."}]
)
```

## Streaming

```python
with client.messages.stream(
    model="claude-opus-4-7",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Tell me a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## Batch Processing

```python
# Create a batch
batch = client.messages.batches.create(
    requests=[
        {"custom_id": "req-1", "params": {"model": "claude-opus-4-7", "max_tokens": 100, "messages": [...]}},
        {"custom_id": "req-2", "params": {"model": "claude-opus-4-7", "max_tokens": 100, "messages": [...]}},
    ]
)

# Poll for results
import time
while True:
    batch = client.messages.batches.retrieve(batch.id)
    if batch.processing_status == "ended":
        break
    time.sleep(60)
```

## Model Migration

When migrating between model versions:

1. Update the `model` string to the new ID
2. Check for deprecated parameters (some older params removed in newer models)
3. Test with the new model — behavior may differ even with the same prompt
4. Update any hardcoded model-specific limits (context window, max output tokens)

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Low cache hit rate | System prompt changes between calls | Make system prompt static; move dynamic content to user turn |
| `anthropic.APIStatusError: 529` | Overloaded | Implement exponential backoff |
| Tool call not executed | Model returned `tool_use` block | Handle `stop_reason: "tool_use"` and call the tool |
| Thinking not appearing | Budget too low | Increase `budget_tokens`; minimum ~1000 |

## Related Skills

- `agent-architecture-audit` — Audit multi-layer agent applications built on the Claude API
- `agent-harness-construction` — Build full agent harnesses using the Claude API
