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
            maxRetries       : 3,
            initialDelay     : 1000,   // ms
            backoffMultiplier: 2,      // 1s, 2s, 4s
            maxDelay         : 30000
        )
    ]
)
```

### GuardrailMiddleware

Block tool calls by name, or by matching their arguments against regex patterns:

```javascript
agent = aiAgent(
    name      : "SafeAgent",
    middleware: [
        new GuardrailMiddleware(
            blockedTools: [ "deleteAllRecords" ],
            argPatterns : { transferFunds: [ { amount: "^[0-9]{6,}$" } ] }
        )
    ]
)
```

{% hint style="info" %}
`GuardrailMiddleware` guards **tool calls**. To filter prompt or response *content*, use the security middleware — see the [Security Guide](../../deployment/security.md).
{% endhint %}

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
    tools       : [ deployTool ],
    checkpointer: aiMemory( "cache" ),
    middleware  : [
        new HumanInTheLoopMiddleware(
            mode                  : "web",
            toolsRequiringApproval: [ "deploy" ]
        )
    ]
)

threadId = "deploy-42"
result   = agent.run( "Deploy the new version to production", {}, { threadId: threadId } )

if ( result.isSuspended() ) {
    pending = result.getData().pendingActions
    // ... notify a human, persist threadId ...

    // Later, after approval:
    finalResponse = agent.resume( "approve", threadId )
}
```

Approval is triggered by **which tool** is being called (or by an `IApprovalPolicy`), not by matching text in the prompt. See [Middleware](../middleware.md) for approval policies, durable grants, and batched approvals.

### FlightRecorderMiddleware

Record the full execution trace for debugging and auditing:

```javascript
recorder = new FlightRecorderMiddleware( mode: "record" )

agent = aiAgent(
    name      : "AuditedAgent",
    middleware: [ recorder ]
)

result = agent.run( "Process this order" )

// Read the recorded tape back off the middleware instance
tape = recorder.getTape()
```

`FlightRecorderMiddleware` runs in one of three modes — `passthrough` (default), `record` (write fixtures to `fixtureDir`), and `replay` (serve responses from `fixturePath`). You can also fetch an attached instance by name with `agent.getMiddlewareByName( "Flight Recorder Middleware" )`.

## Struct-Based Middleware (Inline)

For quick one-off interceptions without creating a class:

```javascript
agent = aiAgent(
    name      : "LoggedAgent",
    middleware: [
        {
            beforeAgentRun: ( context ) => {
                writeLog( "Agent starting: #context.input#" )
                return AiMiddlewareResult.continue()
            },
            afterAgentRun: ( context ) => {
                writeLog( "Agent done: #context.response#" )
                return AiMiddlewareResult.continue()
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
| `AiMiddlewareResult.continue()` | Continue to the next middleware/hook |
| `AiMiddlewareResult.cancel( reason )` | Abort execution |
| `AiMiddlewareResult.suspend( pending )` | Pause execution, checkpoint for resume |
| `AiMiddlewareResult.approve()` | Approve a tool call (used in `beforeToolCall`) |
| `AiMiddlewareResult.reject( reason )` | Skip this tool call; the reason is fed back as its result |
| `AiMiddlewareResult.edit( args )` | Replace the tool call's arguments and run it |
| `AiMiddlewareResult.defer( pending )` | Mark the call as needing a decision, then keep scanning the batch |

See the [full middleware reference](../middleware.md) for every hook, the wrap-style hooks, and the complete result vocabulary.

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
        return AiMiddlewareResult.continue()
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
