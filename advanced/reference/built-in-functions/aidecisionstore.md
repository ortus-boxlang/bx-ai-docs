---
description: Creates an IDecisionStore — the backing store for durable human-approval grants used by Human-in-the-Loop.
icon: database
---

# aiDecisionStore

Creates an `IDecisionStore` — the backing store for durable human-approval grants (`approve_always` and `approve_session`) used by [Human-in-the-Loop](../../../main-components/human-in-the-loop.md).

## Syntax

```javascript
aiDecisionStore( store, config )
```

## Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `store` | `string` | ❌ | `settings.hitl.decisionStore.provider` | `cache`, `jdbc`, `file`, or a full class path |
| `config` | `struct` | ❌ | `settings.hitl.decisionStore.config` | Configuration for the chosen store |

Called with no arguments, it resolves the application-wide default from module settings — the same fallback `HumanInTheLoopMiddleware` uses when it isn't given a store explicitly.

## Returns

An `IDecisionStore` instance.

## Throws

`InvalidDecisionStoreType` for an unrecognised type that isn't a class path.

## Store Types

| Type | Aliases | Backing | Config |
|---|---|---|---|
| `cache` | `CacheDecisionStore` | CacheBox | `cacheName` |
| `jdbc` | `database`, `db`, `JdbcDecisionStore` | Any datasource | `datasource` (required), `table` |
| `file` | `FileDecisionStore` | JSON on disk | `directoryPath` |

## Store Contract

| Method | Returns | Description |
|---|---|---|
| `grant( identity, toolName, scope, expiresAt )` | `void` | Record a grant |
| `isGranted( identity, toolName, scope )` | `boolean` | Whether a live grant exists |
| `revoke( identity, toolName, scope )` | `void` | Remove a grant |
| `listGrants( identity )` | `array` | `[ { toolName, scope, grantedAt, expiresAt } ]` |

## Examples

### Application-Wide Default

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "hitl": {
          "decisionStore": { "provider": "cache", "config": {} }
        }
      }
    }
  }
}
```

```javascript
// Resolves the configured default
store = aiDecisionStore()
```

### Explicit Stores

```javascript
// Database-backed, survives restarts and is shared across nodes
store = aiDecisionStore( "jdbc", { datasource: "myDSN", table: "ai_decisions" } )

// File-backed
store = aiDecisionStore( "file", { directoryPath: "/var/bxai/decisions" } )

// Cache-backed
store = aiDecisionStore( "cache", { cacheName: "default" } )
```

### Attaching to HITL

```javascript
import bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware;

hitl = new HumanInTheLoopMiddleware(
    toolsRequiringApproval: [ "placeOrder" ],
    decisionStore         : aiDecisionStore( "jdbc", { datasource: "myDSN" } )
)
```

### Auditing and Revoking Grants

```javascript
store = aiDecisionStore()

// What has this user permanently allowed?
grants = store.listGrants( "alice@example.com" )

// Take it back
store.revoke( "alice@example.com", "placeOrder" )
```

## Events

Fires `onAiDecisionStoreCreate` with `{ storeType, storeClass, storeConfig }`.

## Related

* [Human-in-the-Loop](../../../main-components/human-in-the-loop.md)
* [Gateways](../../../main-components/gateways.md)
