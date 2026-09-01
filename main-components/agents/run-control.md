---
description: Cancel or steer an AiAgent run already in flight with cancelRun() and steerRun(), addressed purely by threadId — no token to construct or wire up.
icon: gamepad
---

# Agent Run Control

Every `AiAgent` supports cancelling or steering a run already in flight — addressed purely by `threadId`, with nothing extra to construct or thread through your call sites.

## 🚀 Start a Conversation First

Run control is about an **active run**. A typical flow is:

1. Pick or receive a `threadId`.
2. Start the agent run with that `threadId`.
3. Reuse the same `threadId` for follow-up turns.
4. Call `cancelRun()` or `steerRun()` if that run needs intervention.

```javascript
agent = aiAgent(
    name        : "Assistant",
    instructions: "Be concise and helpful"
)

// Pick a stable thread ID for this conversation
threadId = "support-42"

reply1 = agent.run( "Hi, I need help with my invoice", {}, {
    threadId      : threadId,
    userId        : "user-123",
    conversationId: "billing"
} )

// Same conversation thread: reuse the same IDs
reply2 = agent.run( "Can you summarize next steps?", {}, {
    threadId      : threadId,
    userId        : "user-123",
    conversationId: "billing"
} )
```

If you do not pass `options.threadId`, the agent auto-generates one per call. That is great for one-off calls, but you cannot target that run later unless you saved that generated ID.

## 🧵 What Is `threadId`?

`threadId` is the execution correlation key for a run.

* It identifies which in-flight run `cancelRun()` and `steerRun()` should target.
* It links checkpointed/suspended work to the correct thread for resume flows.
* It can be reused across turns to keep a conversation lane consistent.

Think of it as a conversation lane ID, not a message ID.

## 🎯 Core Run-Control Calls

```javascript
agent.cancelRun( threadId )                                       // stop the run
agent.cancelRun( threadId, "user requested cancellation" )        // stop it, with a reason
agent.steerRun( threadId, "actually, focus on the Q3 numbers" )   // splice a message into the live turn
```

## 🎯 How It Works

Both take effect at the run's **next checkpoint** — `beforeLLMCall` or `beforeToolCall` — not instantly. This is deliberate: a run is stopped or redirected between well-defined steps, never mid-flight through a provider call.

* **`cancelRun( threadId, reason = "Cancelled by caller." )`** stops the run with a terminal `AiMiddlewareResult.cancel( reason )`.
* **`steerRun( threadId, input )`** splices a new message into the live request without restarting anything already in progress — nothing already produced is lost.

Both return `false` as a safe no-op when the given thread has no run currently in flight — there's nothing to check before calling either one.

## 🛠️ Practical In-Flight Examples

### Cancel a running task

```javascript
threadId = "research-2026-09-01"

// Start work asynchronously so another code path can cancel it
future = asyncRun( () => {
    return agent.run( "Research every release note and produce a deep report", {}, {
        threadId: threadId
    } )
}, "io-tasks" )

// Later (button click, webhook, admin endpoint, etc.)
wasCancelled = agent.cancelRun( threadId, "User clicked stop" )
```

### Steer a running task without restarting

```javascript
threadId = "analysis-77"

future = asyncRun( () => {
    return agent.run( "Analyze Q3 performance in detail", {}, {
        threadId: threadId
    } )
}, "io-tasks" )

// Nudge the currently running turn at the next checkpoint
wasSteered = agent.steerRun( threadId, "Prioritize revenue and churn; keep output under 10 bullets." )
```

### Steer with a full message struct

```javascript
agent.steerRun( "analysis-77", {
    role   : "user",
    content: "Also include only North America metrics."
} )
```

## ✅ Choosing Good Thread IDs

Use IDs that are stable and unique per active conversation.

* Good: `support-42`, `tenantA-user123-chat9`, `ticket-9981`
* Avoid: random new ID for every turn (unless intentionally one-shot)

If your app is multi-tenant, include tenant/user dimensions in the thread ID or pair it with `userId` and `conversationId` consistently.

## 📡 Events

`onAIAgentRunCancel` / `onAIAgentRunSteer` fire with `{ agent, threadId, reason, input }` — but **only** when the call actually affects a run in flight, never on the no-op case:

```javascript
BoxRegisterInterceptor( ( data ) => {
    log.info( "Run on thread #data.threadId# cancelled: #data.reason#" )
}, "onAIAgentRunCancel" )
```

## 🔌 Powering Gateway Sessions

`aiGatewaySession()`'s `steer` and `interrupt` dispatch policies are built directly on `steerRun()`/`cancelRun()` — a `GatewaySession` calls exactly the same API you can call yourself:

```javascript
session = aiGatewaySession( agent: myAgent, gateways: "http", policy: "interrupt" )
// A message arriving on a busy thread now calls agent.cancelRun() for you, then queues the new one
```

See [Gateway Sessions](../gateway-sessions.md) for the full policy table.

## Related Pages

* [Gateway Sessions](../gateway-sessions.md)
* [Getting Started with Agents](getting-started.md)
* [Streaming](streaming.md)
* [RunControlMiddleware (Internal)](../middleware/run-control.md) — internal cancellation/steering middleware details
* [Middleware Overview](../middleware/README.md) — `AiMiddlewareResult.cancel()` and middleware flow
