---
description: Cancel or steer an AiAgent run already in flight with cancelRun() and steerRun(), addressed purely by threadId — no token to construct or wire up.
icon: gamepad
---

# Agent Run Control

Every `AiAgent` supports cancelling or steering a run already in flight — addressed purely by `threadId`, with nothing extra to construct or thread through your call sites.

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
* [Middleware](../middleware.md) — `AiMiddlewareResult.cancel()`, the mechanism `cancelRun()` uses internally
