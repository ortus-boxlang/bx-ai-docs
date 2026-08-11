---
description: >-
  Events fired when agents, gateways, and decision stores are registered,
  unregistered, or created.
icon: network-wired
---

# Registry & Gateway Events

## Agent Registry Events

### onAIAgentRegistryRegister

Fired when an agent is registered in the global agent registry.

**When**: On `aiAgentRegistry().register()` or `aiAgent( register: true )`

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `agent` | `AiAgent` | The registered agent |
| `key` | string | Registry key (e.g., `agentName@module`) |
| `module` | string | Module namespace |

#### Example

```javascript
BoxRegisterInterceptor( "onAIAgentRegistryRegister", function( event ) {
    println( "Agent registered: #event.key#" )
})
```

***

### onAIAgentRegistryUnregister

Fired when an agent is removed from the global agent registry.

**When**: On `aiAgentRegistry().unregister()` or `unregisterByModule()`

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `key` | string | Registry key |
| `module` | string | Module namespace |

#### Example

```javascript
BoxRegisterInterceptor( "onAIAgentRegistryUnregister", function( event ) {
    println( "Agent unregistered: #event.key#" )
})
```

***

## Gateway Events

Gateways present human-in-the-loop interactions on a platform (CLI, HTTP, or an external module). See the [Security Guide](../../deployment/security/README.md) and [Middleware](../../main-components/middleware.md).

### onGatewayCreate

Fired whenever `aiGateway()` resolves or instantiates a gateway — for core gateways, registry-provided gateways, and direct class paths alike.

| Argument | Type | Description |
| --- | --- | --- |
| `gateway` | `IGateway` | The configured gateway instance |

```javascript
BoxRegisterInterceptor( "onGatewayCreate", function( event ) {
    println( "Gateway ready: #event.gateway.getName()#" )
})
```

### onGatewayRegistryRegister

Fired when a gateway is registered into `gatewayRegistry()` — typically by an external gateway module at load time.

| Argument | Type | Description |
| --- | --- | --- |
| `gateway` | `IGateway` | The gateway being registered |
| `key` | `String` | Registry key (`name` or `name@module`) |
| `module` | `String` | Owning module name, if any |

### onGatewayRegistryUnregister

Fired when a gateway is removed from the registry.

| Argument | Type | Description |
| --- | --- | --- |
| `key` | `String` | Registry key removed |
| `module` | `String` | Module extracted from the key |

***

## Decision Store Events

### onAiDecisionStoreCreate

Fired when `aiDecisionStore()` creates a store for durable human-approval grants (`approve_always` / `approve_session`).

| Argument | Type | Description |
| --- | --- | --- |
| `storeType` | `String` | Requested type (`cache`, `jdbc`, `file`, or a class path) |
| `storeClass` | `String` | Resolved class path |
| `storeConfig` | `Struct` | Configuration passed to the store |

```javascript
BoxRegisterInterceptor( "onAiDecisionStoreCreate", function( event ) {
    println( "Decision store: #event.storeType# -> #event.storeClass#" )
})
```
