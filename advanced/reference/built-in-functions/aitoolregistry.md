# aiToolRegistry

Returns the singleton `AIToolRegistry` instance for registering, discovering, and resolving AI tools globally.

## Syntax

```javascript
aiToolRegistry()
```

## Parameters

No parameters.

## Returns

Returns the singleton `AIToolRegistry` instance. The same registry instance is returned on every call within a runtime.

## Examples

### Register a Tool

```javascript
aiToolRegistry().register(
    name       : "getWeather",
    description: "Get current weather for a location",
    callback   : ( location ) => weatherService.fetch( location )
)
```

### Register with Module Attribution

```javascript
// Keyed as "getWeather@my-module" — easy bulk removal later
aiToolRegistry().register(
    item  : myWeatherTool,
    module: "my-module"
)
```

### Scan a Class for `@AITool` Annotations

```javascript
// Register all methods annotated with @AITool from a class instance
aiToolRegistry().scan( new CustomerService(), "customer-module" )
```

### Scan a Package Path

```javascript
// Scan all .bx files in a package for @AITool-annotated methods
aiToolRegistry().scan( "com.myapp.tools", "my-module" )
```

### Resolve Tools for an Agent

```javascript
// Mix named keys and ITool instances
agent = aiAgent(
    name : "assistant",
    tools: aiToolRegistry().resolveTools( [
        "getWeather",
        "searchProducts@shop-module",
        myCustomToolInstance
    ] )
)
```

### Check and Retrieve Tools

```javascript
registry = aiToolRegistry()

// Check existence
if ( registry.has( "getWeather" ) ) {
    weatherTool = registry.get( "getWeather" )
}

// Get detailed info
info = registry.getToolInfo( "getWeather" )
println( info.name )         // "getWeather"
println( info.description )  // "Get current weather for a location"

// List all registered tools
all = registry.listTools()
all.each( tool => println( tool.getName() ) )

// Get all keys
keys = registry.keys()
println( keys )  // [ "getWeather", "searchProducts@shop-module", ... ]
```

### Unregister Tools

```javascript
// Remove a single tool
aiToolRegistry().unregister( "getWeather" )

// Remove all tools from a module (e.g., on module unload)
aiToolRegistry().unregisterByModule( "my-module" )
```

## Registry API Reference

| Method | Description |
| --- | --- |
| `register( item, module, name, description, callback )` | Register an `ITool` instance or closure |
| `scan( pathOrInstance, module )` | Scan class or package for `@AITool` annotations |
| `scanClass( instance, module )` | Scan a single class instance |
| `get( key )` | Retrieve a tool by key |
| `has( key )` | Check if a key exists |
| `keys()` | Return all registered keys |
| `listTools()` | Return all registered `ITool` instances |
| `getToolInfo( key )` | Return name and description info for a key |
| `resolveTools( array )` | Resolve an array of keys and/or instances to `ITool[]` |
| `unregister( key )` | Remove a tool by key |
| `unregisterByModule( module )` | Remove all tools registered under a module name |

## Related Pages

* [Tool Registry](../../../main-components/tool-registry.md) — Full registry documentation
* [Custom Tools](../../../extending-boxlang-ai/custom-tools.md) — Building custom tool classes
* [aiTool()](aitool.md) — Create `ITool` instances
