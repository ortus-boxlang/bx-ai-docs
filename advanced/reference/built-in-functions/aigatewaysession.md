---
description: Wires one AiAgent to one-or-more IGateway instances for inbound message handling, with a configurable dispatch policy for messages arriving on a busy thread.
icon: satellite-dish
---

# aiGatewaySession

Wires one `AiAgent` to one-or-more `IGateway` instances for inbound message handling. Returns an unstarted `GatewaySession` — call `.start()` to begin accepting inbound messages.

## Syntax

```javascript
aiGatewaySession( agent, gateways, policy, maxQueueDepth, checkpointer )
```

## Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `agent` | `AiAgent` | ✅ | — | The agent every inbound message dispatches to |
| `gateways` | `string` \| `IGateway` \| `array` | ✅ | — | A single gateway or an array. Each entry is either a string name (resolved via `aiGateway()`) or an already-constructed `IGateway` instance |
| `policy` | `string` | ❌ | `"queue"` | `"reject"` \| `"queue"` \| `"steer"` \| `"interrupt"` — how to handle an inbound message on a thread that already has a turn in flight |
| `maxQueueDepth` | `numeric` | ❌ | `50` | Cap on buffered messages per thread for the `"queue"`/`"interrupt"` policies |
| `checkpointer` | `IAiMemory` | ❌ | — | Passed to every gateway's `setCheckpointer()` |

## Returns

An unstarted `GatewaySession` instance.

## Methods

| Method | Returns | Description |
|---|---|---|
| `.start()` | `GatewaySession` | Begin accepting inbound messages on every attached gateway |
| `.stop()` | `GatewaySession` | Stop accepting inbound messages |
| `.isRunning()` | `boolean` | Whether `start()` has been called and `stop()` hasn't since |
| `.getActiveThreadIds()` | `array` | Threads with a turn currently in flight |
| `.getQueueDepth( threadId )` | `numeric` | How many messages are buffered for a thread |

## Examples

### Single Gateway

```javascript
session = aiGatewaySession( agent: myAgent, gateways: "cli" )
session.start()
```

### Multiple Gateways, Steer Policy

```javascript
session = aiGatewaySession(
    agent   : myAgent,
    gateways: [ "cli", aiGateway( "http", { secret: getSecret() } ) ],
    policy  : "steer"
)
session.start()

if ( session.isRunning() ) {
    writeOutput( "Active threads: #arrayLen( session.getActiveThreadIds() )#" )
}
```

## Events

Fires `onGatewaySessionCreate` with `{ session }` when constructed.

## Related

* [Gateway Sessions](../../../main-components/gateway-sessions.md)
* [Agent Run Control](../../../main-components/agents/run-control.md)
* [aiGateway()](aigateway.md) · [aiGatewayRegistry()](aigatewayregistry.md)
