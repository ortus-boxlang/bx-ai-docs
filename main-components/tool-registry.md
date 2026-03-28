---
description: >-
  A centralized registry for discovering, registering, and resolving AI tools
  across modules and classes.
icon: toolbox
---

# Tool Registry

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

The **Tool Registry** is a singleton that manages AI tools across your entire application. Instead of manually passing tool arrays to every agent, you can register tools once and resolve them by name anywhere.

## Why Use the Registry?

| Without Registry | With Registry |
| --- | --- |
| Pass tools array to every agent/model call | Register once, reference by name |
| No cross-module tool sharing | Tools shared across modules |
| Manual `@AITool` discovery | Auto-scan classes for annotated methods |
| Duplicate tool definitions | Single source of truth |

## Accessing the Registry

```javascript
// Get the singleton registry
registry = aiToolRegistry()
```

## Registering Tools

### From an `ITool` Instance

```javascript
myTool = aiTool( "search", "Search the database", ( query ) => db.search( query ) )
aiToolRegistry().register( myTool )
```

### Shorthand Registration

```javascript
// Register a closure directly — no aiTool() wrapper needed
aiToolRegistry().register(
    name       : "calculate",
    description: "Perform math calculations",
    callback   : ( expression ) => evaluate( expression )
)
```

### With Module Attribution

Use a module name to group tools for easy bulk unregistration later:

```javascript
aiToolRegistry().register( myTool, "my-module" )
// Key becomes: "search@my-module"
```

### Scanning a Class with `@AITool`

Annotate methods with `@AITool` and scan the class to register all at once:

```javascript
class CustomerService {

    @AITool( description = "Look up customer by ID or email" )
    function lookupCustomer( required string identifier ) {
        return queryExecute( "SELECT * FROM customers WHERE id = :id OR email = :id", { id: identifier } )
    }

    @AITool( description = "Create a support ticket" )
    function createTicket( required string issue, string priority = "normal" ) {
        return ticketService.create( issue, priority )
    }

    function internalHelper() {  // Not annotated — not registered
        ...
    }
}

// Register all @AITool methods from the class
aiToolRegistry().scan( new CustomerService(), "customer-module" )
```

### Scanning a Package Path

```javascript
// Scan all .bx files in a package for @AITool annotations
aiToolRegistry().scan( "com.myapp.tools", "my-module" )
```

## Using Registered Tools

### Resolve by Name

```javascript
// Get a single tool by key
searchTool = aiToolRegistry().get( "search" )
// or with module:
searchTool = aiToolRegistry().get( "search@my-module" )

// Check existence before use
if ( aiToolRegistry().has( "calculate" ) ) {
    calculatorTool = aiToolRegistry().get( "calculate" )
}
```

### Resolve an Array (Mix of Strings and Instances)

`resolveTools()` lets you pass a heterogeneous array of tool names and/or `ITool` instances:

```javascript
agent = aiAgent(
    name : "Assistant",
    tools: aiToolRegistry().resolveTools( [
        "search",              // Registry key → resolved to ITool
        "calculate@my-module", // Keyed key
        customInlineTool       // ITool instance → passed through
    ] )
)
```

### Introspect the Registry

```javascript
registry = aiToolRegistry()

// List all registered keys
println( registry.keys() )

// Detailed info about a specific tool
info = registry.getToolInfo( "search" )
println( info.name )
println( info.description )

// Full listing
all = registry.listTools()
```

## Unregistering Tools

```javascript
// Remove a single tool
aiToolRegistry().unregister( "search" )

// Remove all tools from a module (bulk cleanup)
aiToolRegistry().unregisterByModule( "my-module" )
```

## Related Pages

* [Tools & MCP (Agents)](agents/tools-and-mcp.md) — Using the registry with agents
* [aiToolRegistry() Reference](../advanced/reference/built-in-functions/aitoolregistry.md) — BIF reference
* [Tools](tools.md) — Core tool concepts and `aiTool()` usage
* [Custom Tools](../extending-boxlang-ai/custom-tools.md) — Building custom tool classes
