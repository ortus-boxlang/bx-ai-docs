---
description: >-
  Attaching middleware to an agent and the agent-specific lifecycle it
  participates in — full hook/result/built-in reference lives on the main
  Middleware page.
icon: filter
---

# Agent Middleware

{% hint style="info" %}
**Since BoxLang AI v3.0+**. This page covers attaching middleware to an **agent** specifically. For the full hook list, the complete `AiMiddlewareResult` vocabulary, every built-in middleware's constructor, and custom middleware classes, see [Middleware](../middleware.md) — that page is the canonical reference.
{% endhint %}

## Adding Middleware to an Agent

Pass an array of middleware instances (or struct-based inline middleware) to `aiAgent()`:

```javascript
agent = aiAgent(
    name      : "SafeAgent",
    middleware: [
        new LoggingMiddleware(),
        new RetryMiddleware( maxRetries: 3 ),
        new GuardrailMiddleware( blockedTools: [ "deleteRecord" ] )
    ]
)
```

Or attach after construction with the fluent API:

```javascript
agent = aiAgent( name: "Assistant" )
    .withMiddleware( new LoggingMiddleware() )
    .withMiddleware( new RetryMiddleware() )
```

Middleware fires **in order** on inbound hooks (`beforeAgentRun`, `beforeLLMCall`, `beforeToolCall`) and in **reverse order** on outbound hooks (`afterToolCall`, `afterLLMCall`, `afterAgentRun`) — a stack, not a flat list. Fetch an attached instance back by name:

```javascript
recorder = agent.getMiddlewareByName( "Flight Recorder Middleware" )
```

## Agent-Scoped Hooks

Two hooks only make sense at the agent level — they bracket the entire `run()` call, not an individual LLM or tool call:

| Hook | Fires When | Context |
| --- | --- | --- |
| `beforeAgentRun` | Agent `run()` begins | `agent`, `input`, `messages`, `params`, `options` |
| `afterAgentRun` | Agent `run()` completes | + `response` |

```javascript
class UsageTrackerMiddleware extends="bxModules.bxai.models.middleware.BaseAiMiddleware" {

    property name="name" default="UsageTracker";

    AiMiddlewareResult function beforeAgentRun( required struct context ) {
        variables.startedAt = getTickCount()
        return AiMiddlewareResult.continue()
    }

    AiMiddlewareResult function afterAgentRun( required struct context ) {
        recordDuration( context.agent.getName(), getTickCount() - variables.startedAt )
        return AiMiddlewareResult.continue()
    }
}
```

Every other hook (`beforeLLMCall`/`afterLLMCall`, `beforeToolCall`/`afterToolCall`, `afterToolBatch`, the wrap-style hooks, `onError`) behaves identically whether attached to an agent or a bare model — see [Middleware](../middleware.md#lifecycle-hooks) for all of them.

## Suspending an Agent for Human Approval

`HumanInTheLoopMiddleware` is the middleware you'll attach to agents most often — it suspends `agent.run()` mid-turn until a human approves, rejects, or edits a pending tool call, and resumes via `agent.resume()`/`resumeStream()`.

```javascript
import bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware;

agent = aiAgent(
    tools       : [ deployTool ],
    checkpointer: aiMemory( "cache" ),
    middleware  : [ new HumanInTheLoopMiddleware(
        mode                  : "web",
        toolsRequiringApproval: [ "deploy" ]
    ) ]
)

threadId = "deploy-42"
result   = agent.run( "Deploy the new version to production", {}, { threadId: threadId } )

if ( result.isSuspended() ) {
    pending = result.getData().pendingActions
    // ... notify a human, persist threadId ...
    finalResponse = agent.resume( "approve", threadId )
}
```

This is a thin slice of a larger topic — approval policies, durable `approve_always`/`approve_session` grants, batched approvals, and presenting through a gateway all live on their own pages:

* [Human-in-the-Loop](../human-in-the-loop.md) — the full picture
* [Agent Memory Management](memory.md) — the `checkpointer` and suspend/resume mechanics
* [Gateways](../gateways.md) — presenting approvals over CLI, HTTP, or a platform module

## Related Pages

* [Middleware](../middleware.md) — full hook reference, `AiMiddlewareResult`, every built-in middleware, custom middleware classes
* [Human-in-the-Loop](../human-in-the-loop.md) — approval policies, durable grants, batching
* [Memory Management](memory.md) — checkpointers and suspend/resume
* [Advanced Patterns](advanced.md) — event interception alternatives
