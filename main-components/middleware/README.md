---
description: Middleware overview and navigation hub for all built-in middleware classes
icon: filter
---

# Middleware Overview

{% hint style="info" %}
Since BoxLang AI v3.0+. This section is the middleware documentation hub: architecture, lifecycle hooks, return semantics, and one dedicated page per middleware class.
{% endhint %}

Middleware wraps agent and model execution in layers. Each layer can observe, mutate, retry, reject, defer, or suspend work around LLM and tool calls without changing business logic.

## Execution Flow

```text
Agent.run(input)
  -> beforeAgentRun (in order)
  -> beforeLLMCall
  -> wrapLLMCall(handler)
  -> afterLLMCall
  -> beforeToolCall (for each tool call)
  -> wrapToolCall(handler)
  -> afterToolCall
  -> afterToolBatch (once per turn)
  -> afterAgentRun (reverse order)
```

Inbound hooks (`before*`) run in registration order.
Outbound hooks (`after*`) run in reverse order.
Wrap hooks (`wrapLLMCall`, `wrapToolCall`) are around-advice and call `handler()` to continue.

## Attach Middleware

```javascript
agent = aiAgent(
    name      : "assistant",
    middleware: [
        new LoggingMiddleware(),
        new RetryMiddleware( maxRetries: 3 ),
        new GuardrailMiddleware( blockedTools: [ "deleteRecord" ] )
    ]
)
```

```javascript
agent = aiAgent( name: "assistant" )
    .withMiddleware( new LoggingMiddleware() )
    .withMiddleware( new RetryMiddleware() )
```

## Hook Reference

| Hook | Fires When | Context Keys |
| --- | --- | --- |
| `beforeAgentRun` | Before an agent run starts | `agent`, `input`, `messages`, `params`, `options` |
| `afterAgentRun` | After an agent run completes | `agent`, `input`, `messages`, `params`, `options`, `response` |
| `beforeLLMCall` | Before each LLM call | `model`, `chatRequest`, `messages` |
| `afterLLMCall` | After each LLM call | `model`, `chatRequest`, `messages`, `response` |
| `beforeToolCall` | Before each tool invocation | `tool`, `toolName`, `toolArgs`, `toolCallId` |
| `afterToolCall` | After each tool invocation | `tool`, `toolName`, `toolArgs`, `toolCallId`, `result` |
| `afterToolBatch` | Once after all tool decisions in a turn | `chatRequest`, `assistantMessage`, `batch` |
| `onError` | On uncaught middleware exceptions | `error`, `phase`, `context` |
| `onAttach` | When attached to an agent/model | attached instance |

## AiMiddlewareResult

Each hook returns `AiMiddlewareResult`.

| Method | Meaning |
| --- | --- |
| `continue()` | Proceed normally |
| `cancel(reason)` | Stop execution |
| `approve()` | Approve a tool call |
| `reject(reason)` | Reject a tool call |
| `edit(args)` | Replace tool args |
| `suspend(pending)` | Suspend run for resume |
| `defer(pending)` | Mark pending and continue batch evaluation |

Predicates include `isContinue()`, `isCancelled()`, `isApproved()`, `isRejected()`, `isEdit()`, `isSuspended()`, `isDeferred()`, and `isTerminal()`.

## Middleware Catalog

### Core Middleware

- [LoggingMiddleware](logging.md)
- [RetryMiddleware](retry.md)
- [GuardrailMiddleware](guardrail.md)
- [MaxToolCallsMiddleware](max-tool-calls.md)
- [HumanInTheLoopMiddleware](human-in-the-loop.md)
- [FlightRecorderMiddleware](flight-recorder.md)
- [RunControlMiddleware (internal)](run-control.md)

### Security Middleware

- [InputSanitizerMiddleware](input-sanitizer.md)
- [OutputGuardMiddleware](output-guard.md)
- [LLMGuardMiddleware](llm-guard.md)

## Related

- [Agent Middleware](../agents/middleware.md)
- [Human-in-the-Loop](../human-in-the-loop.md)
- [Gateways](../gateways.md)
- [Security Guide](../../deployment/security.md)
