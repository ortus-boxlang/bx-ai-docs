---
description: Wire an AiAgent to one or more gateways for inbound message handling with aiGatewaySession() — dispatch policies, queueing, and lifecycle.
icon: satellite-dish
---

# Gateway Sessions

`GatewaySession` (via `aiGatewaySession()`) is the orchestrator that turns "a message arrived on a gateway" into "the agent responded, relayed back through that same gateway" — including deciding what happens when a second message arrives on a thread that already has a turn in flight.

## 🚀 Creating a Session

```javascript
session = aiGatewaySession(
    agent   : myAgent,
    gateways: [ "cli", "http" ],   // single gateway or an array — multiple gateways can share one agent
    policy  : "queue"              // "reject" | "queue" | "steer" | "interrupt"
)
session.start()
```

`gateways` entries can be a string name (resolved via `aiGateway( name )` — core names or anything registered in `aiGatewayRegistry()`) or an already-constructed `IGateway` instance (`aiGateway( "http", { secret: "..." } )` when you need to pass configuration options) — mix and match freely.

## 🧭 Dispatch Policies

A second message arriving on a busy thread is handled per a configurable `policy`:

| Policy | A second message arrives on a busy thread… |
|---|---|
| `reject` | …is refused immediately; the caller must resend. |
| `queue` (default) | …is buffered and dispatched right after the current turn finishes. |
| `steer` | …is spliced into the *currently running* turn via `agent.steerRun()` — not a new turn, nothing already produced is lost. This is a non-destructive splice — **not** the same as some other agent frameworks' "steer," which cancels and restarts. |
| `interrupt` | …asks the current turn to stop via `agent.cancelRun()` (takes effect at its next checkpoint, not instantly), then dispatches the new message next. |

`maxQueueDepth` (default `50`) bounds how many messages can buffer per thread under `queue`/`interrupt` before further messages fall back to an immediate rejection.

## 🪪 Conversation Identity (v3.8+)

Every turn runs with a `conversationId` and, when the platform names the sender, a `userId`, so the agent's memory keeps each chat apart. Before 3.8.0 a turn carried only `{ threadId }`, and every chat on every platform shared one memory bucket.

* The conversation is the chat namespaced by its gateway, for example `telegram:42`. When the platform has threads it adds the thread: `telegram:42:topic-7`, so a Telegram forum topic or a Slack thread is its own conversation.
* `threadId` still names the run: queueing, cancel, steer and checkpoints.
* Conversations kept in the old shared bucket are not found under the new keys.

Subclass `GatewaySession` to change the keys. Both methods are public:

* `resolveConversationId( message, sourceGateway )` returns the conversation id of a message, for example to move a chat to another conversation after a "/new" command.
* `turnOptions( message, sourceGateway, threadId )` returns the options one turn runs with: `threadId`, `conversationId`, and `userId` when the message has one. Override it to name the identity another way, for example with a platform qualified user id.

## 📡 Delivery

Gateways that declare the `"streaming"` capability get chunk-by-chunk delivery via `deliverChunk()`; others get one buffered `deliver()` call once the turn completes. A gateway that pushes inbound messages (rather than being driven by a request/response cycle) implements `IGateway.onMessage()` to register the session's dispatch callback, and `IGateway.onError()` to be notified if its connection drops unexpectedly rather than requiring a caller to poll.

## 🔎 Lifecycle & Observability

```javascript
session.isRunning()                 // has start() been called, and stop() not since?
session.getActiveThreadIds()        // threads with a turn currently in flight
session.getQueueDepth( threadId )   // how many messages are buffered for a thread
```

Every gateway fires interception points on connect/disconnect and inbound/outbound messages — independent of `GatewaySession`, since a gateway can be used directly (e.g. with `HumanInTheLoopMiddleware`) without one:

```javascript
BoxRegisterInterceptor( ( data ) => {
    log.info( "Message on thread #data.threadId# from user #data.userId#" )
}, "onGatewayMessageReceived" )
```

See [Gateways — Events](gateways.md#-events) for the full table.

## Related Pages

* [Gateways](gateways.md) — resolving gateways, core gateways, capabilities, building your own
* [Human-in-the-Loop](human-in-the-loop.md) — approvals presented through a gateway
* [Agent Run Control](agents/run-control.md) — `cancelRun()`/`steerRun()`, the mechanism behind `steer`/`interrupt`
* [aiGatewaySession()](../advanced/reference/built-in-functions/aigatewaysession.md)
