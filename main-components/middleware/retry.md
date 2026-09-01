---
description: Retry failed LLM and tool calls with exponential backoff controls
icon: arrows-rotate
---

# RetryMiddleware

Class: `bxModules.bxai.models.middleware.core.RetryMiddleware`

Wraps both LLM and tool invocations and retries failures with exponential backoff.

## Features

- Retries around `wrapLLMCall`
- Retries around `wrapToolCall`
- Exponential backoff with maximum delay cap
- Skip retries for configured exception types
- Logs retry attempts and terminal failure

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.core.RetryMiddleware(
    maxRetries       : 3,
    initialDelay     : 1000,
    backoffMultiplier: 2,
    maxDelay         : 30000,
    nonRetryableTypes: "InvalidInput,MaxInteractionsExceeded"
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `maxRetries` | numeric | `3` | Retry attempts after initial failure |
| `initialDelay` | numeric | `1000` | First backoff delay in milliseconds |
| `backoffMultiplier` | numeric | `2` | Multiplier for each subsequent delay |
| `maxDelay` | numeric | `30000` | Maximum backoff delay in milliseconds |
| `nonRetryableTypes` | string | `"InvalidInput,MaxInteractionsExceeded"` | Comma-delimited exception types to bypass retries |

## Hooks Used

- `wrapLLMCall`
- `wrapToolCall`

## Example

```javascript
agent = aiAgent(
    name      : "resilient-agent",
    middleware: [
        new bxModules.bxai.models.middleware.core.RetryMiddleware(
            maxRetries       : 5,
            initialDelay     : 500,
            backoffMultiplier: 1.5,
            maxDelay         : 10000,
            nonRetryableTypes: "InvalidInput,BXAI.SecurityViolation"
        )
    ]
)
```

## Notes

- Retries are applied to exceptions only; successful responses are returned immediately.
- Non-retryable exception types are matched case-insensitively via list lookup.
