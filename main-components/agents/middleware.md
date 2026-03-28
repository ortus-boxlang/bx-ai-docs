---
description: >-
  Middleware hooks let you intercept agent lifecycle events for logging,
  retry, guardrails, tool control, and human-in-the-loop patterns.
icon: filter
---

# Agent Middleware

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

Middleware lets you intercept and control an agent's execution at key lifecycle points — before/after each agent run, before/after each LLM call, and before/after each tool invocation. This enables logging, retry logic, content guardrails, rate limiting, and human approval workflows without modifying agent logic.

## Adding Middleware to an Agent

```javascript
agent = aiAgent(
    name      : "SafeAgent",
    middleware: [
        new LoggingMiddleware(),
        new RetryMiddleware( maxRetries: 3 ),
        new GuardrailMiddleware()
    ]
)
```

Middleware fires **in order** on inbound hooks (`beforeAgentRun`, `beforeLLMCall`, `beforeToolCall`) and in **reverse order** on outbound hooks (`afterToolCall`, `afterLLMCall`, `afterAgentRun`).

## Built-In Middleware

### LoggingMiddleware

Logs lifecycle events to the BoxLang `ai` log and optionally to console:

```javascript
agent = aiAgent(
    name      : "TracedAgent",
    middleware: [
        new LoggingMiddleware(
            logToFile   : true,
            logToConsole: true,
            logLevel    : "info",
            prefix      : "[Support Bot]"
        )
    ]
)
```

### RetryMiddleware

Automatically retries failed LLM calls with exponential backoff:

```javascript
agent = aiAgent(
    name      : "ResilientAgent",
    middleware: [
        new RetryMiddleware(
            maxRetries  : 3,
            delayMs     : 1000,
            backoffFactor: 2.0   // 1s, 2s, 4s
        )
    ]
)
```

### GuardrailMiddleware

Block, filter, or rewrite requests and responses based on content rules:

```javascript
agent = aiAgent(
    name      : "SafeAgent",
    middleware: [
        new GuardrailMiddleware(
            blockedPhrases: [ "competitor_name", "confidential" ],
            maxInputLength : 5000,
            onBlock        : ( context ) => "I can't help with that topic."
        )
    ]
)
```

### MaxToolCallsMiddleware

Prevent runaway tool-call loops:

```javascript
agent = aiAgent(
    name      : "BoundedAgent",
    middleware: [
        new MaxToolCallsMiddleware( maxCalls: 10 )
    ]
)
```

### HumanInTheLoopMiddleware

Suspend the agent mid-run for human approval before continuing:

```javascript
agent = aiAgent(
    name        : "ApprovalAgent",
    checkpointer: aiMemory( "cache" ),
    middleware  : [
        new HumanInTheLoopMiddleware(
            triggerPhrase: "deploy to production"
        )
    ]
)

result = agent.run( "Deploy the new version to production" )

if ( result.isSuspended() ) {
    threadId = result.getThreadId()
    // ... notify human, store threadId ...

    // Later, after approval:
    finalResponse = agent.resume(
        decision: "approved",
        threadId: threadId
    )
}
```

### FlightRecorderMiddleware

Record the full execution trace for debugging and auditing:

```javascript
agent = aiAgent(
    name      : "AuditedAgent",
    middleware: [
        new FlightRecorderMiddleware()
    ]
)

result      = agent.run( "Process this order" )
flightRecord = agent.getMiddleware( "FlightRecorder" ).getRecord()

flightRecord.each( event => {
    println( "#event.phase# at #event.timestamp#: #event.summary#" )
} )
```

## Struct-Based Middleware (Inline)

For quick one-off interceptions without creating a class:

```javascript
agent = aiAgent(
    name      : "LoggedAgent",
    middleware: [
        {
            beforeAgentRun: ( context ) => {
                writeLog( "Agent starting: #context.input#" )
                return AiMiddlewareResult::proceed()
            },
            afterAgentRun: ( context ) => {
                writeLog( "Agent done: #context.response#" )
                return AiMiddlewareResult::proceed()
            }
        }
    ]
)
```

## Adding Middleware After Construction

```javascript
agent = aiAgent( name: "Assistant" )
    .withMiddleware( new LoggingMiddleware() )
    .withMiddleware( new RetryMiddleware() )
```

## Middleware Result Actions

Each hook returns an `AiMiddlewareResult` that controls execution flow:

| Result | Effect |
|---|---|
| `AiMiddlewareResult::proceed()` | Continue to the next middleware/hook |
| `AiMiddlewareResult::cancel( message )` | Abort execution, return message |
| `AiMiddlewareResult::suspend( state )` | Pause execution, save state for resume |
| `AiMiddlewareResult::approve()` | Approve a tool call (used in `beforeToolCall`) |
| `AiMiddlewareResult::reject( reason )` | Reject a tool call |

## Lifecycle Hooks

| Hook | Fires When | Context Available |
|---|---|---|
| `beforeAgentRun` | Agent `run()` begins | `agent`, `input`, `messages`, `params`, `options` |
| `afterAgentRun` | Agent `run()` completes | + `response` |
| `beforeLLMCall` | Each HTTP call to AI provider | `model`, `chatRequest`, `messages` |
| `afterLLMCall` | Each HTTP call completes | + `response` |
| `beforeToolCall` | Each tool invocation | `tool`, `toolName`, `toolArgs`, `toolCallId` |
| `afterToolCall` | Each tool invocation completes | + `result` |
| `onError` | Any hook throws an exception | `error`, `phase`, `context` |

## Custom Middleware

Extend `BaseAiMiddleware` and override only the hooks you need:

```javascript
import bxModules.bxai.models.middleware.BaseAiMiddleware;
import bxModules.bxai.models.middleware.AiMiddlewareResult;

class CostTrackerMiddleware extends="BaseAiMiddleware" {

    property name="name" default="CostTracker";

    AiMiddlewareResult function afterLLMCall( required struct context ) {
        var usage = context.response?.usage
        if ( !isNull( usage ) ) {
            billingService.record(
                model  : context.model.getProvider(),
                tokens : usage.total_tokens
            )
        }
        return AiMiddlewareResult::proceed()
    }
}

// Use it
agent = aiAgent(
    name      : "BilledAgent",
    middleware: [ new CostTrackerMiddleware() ]
)
```

## Related Pages

* [Middleware](../middleware.md) — Full middleware documentation and all built-in types
* [Memory Management](memory.md) — Using checkpointer for suspend/resume
* [Advanced Patterns](advanced.md) — Event interception alternatives
