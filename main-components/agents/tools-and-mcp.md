---
description: >-
  Tools, the Global Tool Registry, MCP server seeding, and the ClosureTool
  pattern for building AI agents with real-world capabilities.
icon: wrench
---

# Agent Tools & MCP

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

Agents use tools to perform real-world actions — querying databases, calling APIs, running calculations, and more. In v3.0, tools can also be sourced from the Global Tool Registry and from remote MCP servers.

## Basic Tool Usage

```javascript
// Inline lambda tool
weatherTool = aiTool(
    "get_weather",
    "Get current weather for a location",
    location => getWeatherData( location )
).describeLocation( "City and country, e.g. Boston, MA" )

agent = aiAgent(
    name : "WeatherBot",
    tools: [ weatherTool ]
)

response = agent.run( "What's the weather in Paris?" )
// Agent automatically calls get_weather("Paris")
```

## Global Tool Registry (v3.0+)

The Tool Registry is a module-wide singleton that stores tools by name (optionally namespaced by module). Agents can look up tools by name at runtime instead of holding direct references.

```javascript
// Register tools centrally
aiToolRegistry()
    .register( weatherTool, module: "my-app" )
    .register( calculatorTool )

// Resolve tools by name when creating agents
agent = aiAgent(
    name : "Assistant",
    tools: aiToolRegistry().resolveTools( [
        "get_weather@my-app",
        "calculate"
    ] )
)
```

### Registering with Shorthand

```javascript
// Register directly from function reference
aiToolRegistry().register(
    name       : "search_products",
    description: "Search the product catalog",
    callback   : searchProducts
)
```

### `@AITool` Annotation Scanning

Annotate methods with `@AITool` and scan a class or package path:

```javascript
class MyTools {
    @AITool( "Search the product catalog" )
    function search_products( required string query ) {
        return queryProductDB( query )
    }

    @AITool( { name: "get_order", description: "Look up an order by ID" } )
    function get_order( required string orderId ) {
        return queryOrderDB( orderId )
    }
}

// Scan and register all @AITool methods at startup
aiToolRegistry().scan( new MyTools(), module: "my-app" )

// Use in agent
agent = aiAgent(
    name : "SalesBot",
    tools: aiToolRegistry().resolveTools( [ "search_products@my-app", "get_order@my-app" ] )
)
```

### Registry API

| Method | Description |
|---|---|
| `register( tool, module )` | Register an `ITool` instance |
| `register( name, description, callback, module )` | Shorthand — builds `ClosureTool` |
| `scan( instanceOrPath, module )` | Scan `@AITool` annotations |
| `get( key )` | Get a tool by name or `name@module` key |
| `has( key )` | Check if a tool exists |
| `resolveTools( array )` | Convert string keys to `ITool` instances |
| `unregister( key )` | Remove a tool |
| `keys()` | List all registered keys |

See [Tool Registry](../tool-registry.md) for the full reference.

## ClosureTool Pattern

`ClosureTool` wraps a lambda directly without going through `aiTool()`:

```javascript
import bxModules.bxai.models.tools.ClosureTool;

searchTool = new ClosureTool(
    "search_docs",
    "Search documentation",
    query => searchIndex( query )
)

agent = aiAgent(
    name : "DocBot",
    tools: [ searchTool ]
)
```

## Accessing `_chatRequest` in Tools

Tools receive the live `AiChatRequest` as `_chatRequest` in their arguments — useful for reading the current model, adding context, or adjusting behavior:

```javascript
contextTool = aiTool(
    "log_context",
    "Log information about the current request",
    ( args ) => {
        model   = args._chatRequest.getModel()
        history = args._chatRequest.getMessages()
        writeLog( "Tool called during #model# with #history.len()# messages" )
        return "Logged"
    }
)
```

## MCP Server Seeding (v3.0+)

Connect agents to remote MCP servers. Tool names from the server are automatically fetched and made available to the agent:

```javascript
agent = aiAgent(
    name      : "ResearchAgent",
    mcpServers: [
        {
            url      : "http://localhost:3000/mcp",
            toolNames: [ "web_search", "fetch_page" ]
        },
        {
            url      : "http://internal-tools/mcp",
            toolNames: [ "query_crm" ]
        }
    ]
)

// Agent can use web_search, fetch_page, and query_crm from the MCP servers
response = agent.run( "Find recent news about BoxLang" )
```

### MCP Authentication

```javascript
agent = aiAgent(
    name      : "SecureAgent",
    mcpServers: [
        {
            url        : "https://tools-server/mcp",
            toolNames  : [ "search" ],
            bearerToken: server.bearerToken,
            headers    : { "X-Tenant-ID": "acme" }
        }
    ]
)
```

## Dynamic Tool Assignment

```javascript
function getToolsForRole( required string userRole ) {
    if ( userRole == "admin" ) {
        return aiToolRegistry().resolveTools( [ "admin_tool", "user_tool", "report_tool" ] )
    }
    return aiToolRegistry().resolveTools( [ "user_tool" ] )
}

agent = aiAgent( name: "ContextAgent" )
    .setTools( getToolsForRole( currentUser.role ) )
```

## Related Pages

* [Tool Registry](../tool-registry.md) — Full registry reference
* [AI Tools](../tools.md) — `aiTool()` BIF reference
* [Skills](skills.md) — Injecting knowledge vs. calling tools
* [MCP Server](../../advanced/mcp-server.md) — Building your own MCP server
* [MCP Client](../../advanced/mcp-client.md) — Consuming MCP servers directly
