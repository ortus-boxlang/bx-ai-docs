# aiGateway

Resolves an `IGateway` instance by name — a gateway bx-ai ships in core, or one registered by an external gateway module.

## Syntax

```javascript
aiGateway( name, options )
```

## Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | `string` | ✅ | — | A core gateway name (`mock`, `cli`, `http`), a name registered in `gatewayRegistry()`, or a full class path |
| `options` | `struct` | ❌ | `{}` | Configuration passed to the gateway's `configure()`. Shape is gateway-specific |

## Returns

A configured `IGateway` instance.

## Throws

`GatewayNotSupported` when the name matches no core gateway, nothing is registered under it, and it is not an instantiable class path.

## Resolution Order

1. **Core gateways** — `mock`, `cli`, `http`
2. **Registered gateways** — anything a module put in `gatewayRegistry()`
3. **Class path** — the name is tried as a directly-instantiable class
4. Otherwise `GatewayNotSupported`

## Examples

### Core Gateways

```javascript
// Blocking CLI approval prompt
cli = aiGateway( "cli" )

// Signed, network-reachable webhooks
http = aiGateway( "http", {
    secret           : getSecret(),
    callbackUrl      : "https://ops.example.com/bxai-events",
    toleranceSeconds : 300,
    requestTTLSeconds: 900
} )

// In-memory gateway for tests
mock = aiGateway( "mock" )
```

### Presenting Approvals Through a Gateway

```javascript
import bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware;

agent = aiAgent(
    tools       : [ deployTool ],
    middleware  : [ new HumanInTheLoopMiddleware(
        toolsRequiringApproval: [ "deploy" ],
        gateway               : aiGateway( "http", { secret: getSecret() } )
    ) ],
    checkpointer: aiMemory( "cache" )
)
```

### An Externally-Registered Gateway

```javascript
// The module registers itself when it loads
gatewayRegistry().register( new MyPlatformGateway(), "bx-ai-gateway-myplatform" )

// Resolve it by name anywhere
gateway = aiGateway( "myplatform" )
```

### Checking Capabilities

```javascript
gateway = aiGateway( "http" )

if ( gateway.supports( "humanApproval" ) ) {
    // safe to present an approval request
}

gateway.getCapabilities()  // [ "inboundMessages", "outboundMessages", "humanApproval" ]
```

## Events

Fires `onGatewayCreate` with `{ gateway }` on every resolution path.

## Related

* [Gateways](../../../main-components/gateways.md)
* [Human-in-the-Loop](../../../main-components/human-in-the-loop.md)
* [gatewayRegistry()](gatewayregistry.md)
