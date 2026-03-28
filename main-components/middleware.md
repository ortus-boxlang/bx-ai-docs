---
description: >-
  Intercept, modify, log, retry, and guard agent execution at every stage using
  the middleware pipeline.
icon: filter
---

# Middleware

Middleware provides hooks into every stage of agent execution — before and after LLM calls, tool invocations, and the full agent run. Use it for logging, retrying failures, enforcing guardrails, and more without touching your agent code.

## How It Works

Middleware wraps agent execution in layers. Each layer can inspect and modify the request/response, or halt execution entirely.

```
Agent.run(input)
  │
  ▼ beforeAgentRun (all middleware, in order)
  │
  ▼ beforeLLMCall → (LLM call) → afterLLMCall
  │
  ▼ beforeToolCall → (tool execution) → afterToolCall
  │
  ▼ afterAgentRun (all middleware, in reverse order)
  │
  ▼ result
```

**Inbound hooks** (before\*) run in registration order.
**Outbound hooks** (after\*) run in reverse order.

## Adding Middleware to an Agent

```javascript
agent = aiAgent(
    name      : "assistant",
    middleware: [
        new LoggingMiddleware(),
        new RetryMiddleware( maxRetries: 3 ),
        new GuardrailMiddleware( { blockedTools: [ "deleteRecord" ] } )
    ]
)
```

Or with the fluent API:

```javascript
agent = aiAgent( name: "assistant" )
    .withMiddleware( new LoggingMiddleware() )
    .withMiddleware( new RetryMiddleware() )
```

## Lifecycle Hooks

| Hook | Fires When | Context Keys |
| --- | --- | --- |
| `beforeAgentRun` | Before agent starts processing | `agent`, `input`, `messages`, `params`, `options` |
| `afterAgentRun` | After agent finishes | `agent`, `input`, `messages`, `params`, `options`, `response` |
| `beforeLLMCall` | Before each LLM API call | `model`, `chatRequest`, `messages` |
| `afterLLMCall` | After each LLM API call | `model`, `chatRequest`, `messages`, `response` |
| `beforeToolCall` | Before each tool execution | `tool`, `toolName`, `toolArgs`, `toolCallId` |
| `afterToolCall` | After each tool execution | `tool`, `toolName`, `toolArgs`, `toolCallId`, `result` |
| `onError` | On any exception | `error`, `phase` (hook name), `context` (hook context) |

## Middleware Return Values (`AiMiddlewareResult`)

Each hook returns an `AiMiddlewareResult` to control the flow:

| Factory Method | Effect |
| --- | --- |
| `AiMiddlewareResult::continue()` | Proceed normally (default) |
| `AiMiddlewareResult::cancel( reason )` | Stop execution, return reason as error |
| `AiMiddlewareResult::approve()` | Explicitly approve (used in Human-in-the-Loop) |
| `AiMiddlewareResult::reject( reason )` | Reject with explanation |
| `AiMiddlewareResult::edit( args )` | Modify tool arguments before execution |
| `AiMiddlewareResult::suspend( pending )` | Pause execution (async Human-in-the-Loop) |

## Built-in Middleware

BoxLang AI ships with six battle-tested middleware classes covering the most common cross-cutting concerns. All live in `bxModules.bxai.models.middleware.core`.

| Middleware | When to Use It |
| --- | --- |
| `LoggingMiddleware` | Audit every LLM call and tool invocation — write to console, file, or both with a configurable log level |
| `RetryMiddleware` | Automatically retry failed LLM calls with exponential back-off; essential for flaky or rate-limited providers |
| `GuardrailMiddleware` | Block dangerous tools entirely and enforce regex-based argument validation before any tool runs |
| `MaxToolCallsMiddleware` | Prevent runaway agents by capping the total number of tool invocations per run |
| `HumanInTheLoopMiddleware` | Require explicit human approval (CLI prompt or custom callback) before sensitive tools execute |
| `FlightRecorderMiddleware` | Record live LLM/tool interactions to a JSON fixture and replay them offline — ideal for testing and debugging |

### LoggingMiddleware

Logs every agent lifecycle event.

```javascript
middleware = new bxModules.bxai.models.middleware.core.LoggingMiddleware(
    logToFile      : true,
    logToConsole   : false,
    logLevel       : "info",     // "info", "warn", "error", "debug"
    prefix         : "[AI Middleware]"
)
```

### RetryMiddleware

Retries failed LLM calls with exponential backoff.

```javascript
middleware = new bxModules.bxai.models.middleware.core.RetryMiddleware(
    maxRetries         : 3,
    initialDelay       : 1000,      // ms
    backoffMultiplier  : 2,
    maxDelay           : 30000,
    nonRetryableTypes  : "InvalidInput,MaxInteractionsExceeded"
)
```

### GuardrailMiddleware

Blocks specified tools and validates tool arguments against regex patterns.

```javascript
middleware = new bxModules.bxai.models.middleware.core.GuardrailMiddleware(
    blockedTools : [ "deleteRecord", "dropTable" ],
    argPatterns  : {
        "runSQL": { "query": "^SELECT" }  // Only allow SELECT statements
    }
)
```

### MaxToolCallsMiddleware

Caps the total number of tool invocations in a single run.

```javascript
middleware = new bxModules.bxai.models.middleware.core.MaxToolCallsMiddleware(
    maxCalls: 10
)
```

### HumanInTheLoopMiddleware

Requires human approval before specified tools execute.

```javascript
// CLI mode — blocks until user types y/n
middleware = new bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware(
    toolsRequiringApproval: [ "sendEmail", "chargeCard" ],
    mode                  : "cli",    // "cli" or "web"
    showArguments         : true
)

// Callback mode — custom approval logic
middleware = new bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware(
    toolsRequiringApproval: [ "sendEmail" ],
    approvalCallback      : ( toolName, toolArgs ) => {
        return auditSystem.promptApproval( toolName, toolArgs )
    }
)
```

### FlightRecorderMiddleware

Records LLM and tool interactions to a JSON fixture for debugging and replay.

```javascript
// Passthrough mode (observe only, no writing)
middleware = new bxModules.bxai.models.middleware.core.FlightRecorderMiddleware(
    mode: "passthrough"
)

// Record mode — captures interactions to disk
middleware = new bxModules.bxai.models.middleware.core.FlightRecorderMiddleware(
    mode       : "record",
    fixtureDir : ".ai/flight-recorder",
    recordTools: true
)

// Replay mode — returns fixture data without live LLM calls (great for testing)
middleware = new bxModules.bxai.models.middleware.core.FlightRecorderMiddleware(
    mode       : "replay",
    fixturePath: ".ai/flight-recorder/test-run.json",
    strict     : true    // Error if interaction not found in fixture
)
```

## Struct-Based Inline Middleware

For simple cases, pass a struct with hook functions — no class required:

```javascript
agent = aiAgent(
    name      : "assistant",
    middleware: [
        {
            beforeLLMCall: function( context ) {
                println( "Calling LLM with #context.messages.len()# messages" )
                return AiMiddlewareResult::continue()
            },
            afterLLMCall: function( context ) {
                println( "Got response: #context.response.getContent().left(80)#..." )
                return AiMiddlewareResult::continue()
            }
        }
    ]
)
```

## Custom Middleware Class

Extend `BaseAiMiddleware` for reusable, configurable middleware:

```javascript
class extends="bxModules.bxai.models.middleware.BaseAiMiddleware" {

    property name="maxTokensPerCall" type="numeric" default=4000;

    function init( numeric maxTokensPerCall = 4000 ) {
        variables.maxTokensPerCall = arguments.maxTokensPerCall
        variables.name             = "Token Budget Middleware"
        variables.description      = "Cancels requests that would exceed the token budget"
        return this
    }

    function beforeLLMCall( required struct context ) {
        var estimated = context.messages.reduce( ( acc, msg ) => acc + msg.content.len() / 4, 0 )
        if ( estimated > variables.maxTokensPerCall ) {
            return AiMiddlewareResult::cancel( "Estimated token count #estimated# exceeds budget of #variables.maxTokensPerCall#" )
        }
        return AiMiddlewareResult::continue()
    }
}
```

## Combining Middleware

Stack middleware to compose behaviors:

```javascript
agent = aiAgent(
    name      : "production-agent",
    middleware: [
        new LoggingMiddleware( logToConsole: false, logToFile: true ),
        new RetryMiddleware( maxRetries: 3 ),
        new MaxToolCallsMiddleware( maxCalls: 20 ),
        new GuardrailMiddleware( blockedTools: [ "deleteUser" ] ),
        new FlightRecorderMiddleware( mode: "passthrough" )
    ]
)
```

## Related Pages

* [Agents — Middleware](agents/middleware.md) — Agent-specific middleware patterns
* [Custom Tools](../extending-boxlang-ai/custom-tools.md) — Build tools that middleware can intercept
