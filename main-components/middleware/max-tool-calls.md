---
description: Enforce a strict per-run cap on tool calls to prevent runaway loops
icon: hand
---

# MaxToolCallsMiddleware

Class: `bxModules.bxai.models.middleware.core.MaxToolCallsMiddleware`

Caps the number of tool calls in one agent run. Useful as a hard safety boundary.

## Features

- Tracks a per-run call counter
- Resets counter on each new run
- Cancels execution when call cap is exceeded

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.core.MaxToolCallsMiddleware(
    maxCalls: 10
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `maxCalls` | numeric | `10` | Maximum tool calls allowed per agent run |

## Hooks Used

- `beforeAgentRun` (counter reset)
- `beforeToolCall` (counter increment and cap enforcement)

## Example

```javascript
agent = aiAgent(
    name      : "bounded-agent",
    middleware: [
        new bxModules.bxai.models.middleware.core.MaxToolCallsMiddleware( maxCalls: 5 )
    ]
)
```

## Notes

- Exceeding the cap returns `AiMiddlewareResult.cancel(...)`.
- Combine with `RetryMiddleware` and `GuardrailMiddleware` for stronger control of autonomous loops.
