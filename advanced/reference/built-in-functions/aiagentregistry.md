# aiAgentRegistry

Get the singleton global `AIAgentRegistry` instance for registering, resolving, and observing agents.

## Syntax

```javascript
aiAgentRegistry()
```

## Parameters

None.

## Returns

`AIAgentRegistry` singleton instance.

## Key Format

Agents are keyed as:

- `agentName`
- `agentName@moduleName`

## Core Methods

| Method | Description |
|---|---|
| `register( agent, module = "" )` | Register an `AiAgent` instance |
| `get( key )` | Retrieve agent by key |
| `has( key )` | Check whether key exists |
| `unregister( key )` | Remove agent by key |
| `unregisterByModule( module )` | Remove all agents in a module namespace |
| `resolveAgents( agents )` | Resolve mixed key/instance arrays to `AiAgent[]` |
| `getAgentInfo( key )` | Lightweight metadata: name, description, module |
| `listAgents()` | Return all registered agents with metadata |
| `keys()` | List all keys |
| `clear()` | Remove all entries |

## Events Fired

| Event | When |
|---|---|
| `onAIAgentRegistryRegister` | After successful registration |
| `onAIAgentRegistryUnregister` | After unregister |

## Examples

### Register and fetch

```javascript
agent = aiAgent(
    name: "support-agent",
    description: "Handles customer support"
)

aiAgentRegistry().register( agent, "my-app" )

saved = aiAgentRegistry().get( "support-agent@my-app" )
```

### Resolve mixed arrays

```javascript
resolved = aiAgentRegistry().resolveAgents( [
    "support-agent@my-app",
    anotherAgent
] )
```

### List observability metadata

```javascript
info = aiAgentRegistry().listAgents()
println( jsonSerialize( info ) )
```

### Unregister by module

```javascript
aiAgentRegistry().unregisterByModule( "my-app" )
```
