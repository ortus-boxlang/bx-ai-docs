---
description: Returns the singleton GatewayRegistry — the extension point external gateway modules use to make their gateway resolvable by aiGateway().
icon: plug
---

# aiGatewayRegistry

Returns the singleton `GatewayRegistry` — the extension point external gateway modules use to make their gateway resolvable by `aiGateway()`.

## Syntax

```javascript
aiGatewayRegistry()
```

## Parameters

No parameters.

## Returns

The singleton `GatewayRegistry` instance. The same instance is returned on every call within a runtime.

## Methods

| Method | Returns | Description |
|---|---|---|
| `register( gateway, module )` | `GatewayRegistry` | Register a gateway; `module` namespaces the key |
| `unregister( key )` | `GatewayRegistry` | Remove one gateway by key |
| `unregisterByModule( module )` | `GatewayRegistry` | Remove every gateway a module registered |
| `has( key )` | `boolean` | Whether a key is registered |
| `get( key )` | `IGateway` | Fetch a registered gateway |
| `listGateways()` | `struct` | Every registered gateway, keyed |

Keys follow the `name` or `name@module` convention, so two modules can each provide a gateway of the same name without colliding.

## Throws

`InvalidArgument` if the registered object is not an `IGateway`.

## Examples

### Registering From a Module

```javascript
// In your gateway module's ModuleConfig onLoad()
function onLoad() {
    aiGatewayRegistry().register( new MyPlatformGateway(), "bx-ai-gateway-myplatform" )
}

function onUnload() {
    aiGatewayRegistry().unregisterByModule( "bx-ai-gateway-myplatform" )
}
```

Once registered, the gateway resolves by name:

```javascript
gateway = aiGateway( "myplatform" )
```

### Inspecting the Registry

```javascript
if ( aiGatewayRegistry().has( "myplatform" ) ) {
    gateway = aiGatewayRegistry().get( "myplatform" )
}

// Fully-qualified key
gateway = aiGatewayRegistry().get( "myplatform@bx-ai-gateway-myplatform" )

// Everything currently registered
registered = aiGatewayRegistry().listGateways()
```

## Events

| Event | Payload |
|---|---|
| `onGatewayRegistryRegister` | `gateway`, `key`, `module` |
| `onGatewayRegistryUnregister` | `key`, `module` |

## Related

* [Gateways](../../../main-components/gateways.md)
* [aiGateway()](aigateway.md)
