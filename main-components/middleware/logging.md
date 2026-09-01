---
description: Audit lifecycle events with configurable file and console logging
icon: file-lines
---

# LoggingMiddleware

Class: `bxModules.bxai.models.middleware.core.LoggingMiddleware`

Logs agent, LLM, and tool lifecycle events to the `ai` log and optionally to stdout.

## Features

- Logs `beforeAgentRun` and `afterAgentRun`
- Logs `beforeLLMCall` and `afterLLMCall`
- Logs `beforeToolCall` and `afterToolCall`
- Logs `onError` with phase and error message
- Supports log prefix and log level

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.core.LoggingMiddleware(
    logToFile    : true,
    logToConsole : false,
    logLevel     : "info",
    prefix       : "[AI Middleware]"
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `logToFile` | boolean | `true` | Write events to BoxLang log `ai` |
| `logToConsole` | boolean | `false` | Also print events to stdout |
| `logLevel` | string | `"info"` | Base level for emitted messages |
| `prefix` | string | `"[AI Middleware]"` | Prefix prepended to each line |

## Hooks Used

- `beforeAgentRun`
- `afterAgentRun`
- `beforeLLMCall`
- `afterLLMCall`
- `beforeToolCall`
- `afterToolCall`
- `onError`

## Example

```javascript
agent = aiAgent(
    name      : "audited-agent",
    middleware: [
        new bxModules.bxai.models.middleware.core.LoggingMiddleware(
            logToFile   : true,
            logToConsole: true,
            logLevel    : "debug",
            prefix      : "[Agent Audit]"
        )
    ]
)
```

## Notes

- `onError` logs with level `error` and then returns `continue()`.
- Message content is truncated in emitted text for readability.
