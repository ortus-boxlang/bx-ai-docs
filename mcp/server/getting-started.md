---
icon: rocket
---

# Getting Started with MCP Servers

Get your first MCP server running in 5 minutes.

## What You'll Build

A simple MCP server that exposes two tools: search and calculate. Clients can invoke these tools via the Model Context Protocol.

## Step 1: Register Tools at Application Start

Create or edit your `Application.bx` to register tools when the application starts:

```javascript
// Application.bx
class {

    function onApplicationStart() {
        // Get or create an MCP server instance
        MCPServer( "myApp" )
            .setDescription( "My Application MCP Server" )
            .setVersion( "1.0.0" )

            // Register first tool
            .registerTool(
                aiTool( "search", "Search for documents", ( query ) => {
                    return searchService.search( query )
                } )
            )

            // Register second tool
            .registerTool(
                aiTool( "calculate", "Perform calculations", ( expression ) => {
                    return evaluate( expression )
                } )
            )
    }

    function onApplicationEnd() {
        // Clean up on shutdown
        bxModules.bxai.models.mcp.MCPServer::removeInstance( "myApp" )
    }

}
```

## Step 2: Access the MCP Endpoint

The module provides a built-in HTTP endpoint at `public/mcp.bxm`. Make a request specifying your server:

```bash
# Using query parameter
POST http://localhost/~bxai/mcp.bxm?server=myApp
Content-Type: application/json

{"jsonrpc":"2.0","method":"tools/list","id":"1"}
```

Or use URL segment:

```bash
# Using URL segment
POST http://localhost/~bxai/mcp.bxm/myApp
```

## Step 3: List Your Tools

```bash
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'
```

Response:

```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": {
        "tools": [
            {
                "name": "search",
                "description": "Search for documents",
                "inputSchema": { ... }
            },
            {
                "name": "calculate",
                "description": "Perform calculations",
                "inputSchema": { ... }
            }
        ]
    }
}
```

## Step 4: Invoke a Tool

```bash
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "id": "2",
    "params": {
      "name": "search",
      "arguments": { "query": "BoxLang" }
    }
  }'
```

Response:

```json
{
    "jsonrpc": "2.0",
    "id": "2",
    "result": {
        "content": [
            { "type": "text", "text": "Search results for 'BoxLang'..." }
        ]
    }
}
```

## What's Next?

- 📖 [Transports](transports.md) — HTTP vs STDIO setup
- 🔐 [Server Configuration](server-configuration.md) — Add authentication
- 🧩 [Registering Tools](registration.md) — More complex tools
- 📋 [Observability](observability.md) — Monitor your server
- 🏗️ [Class-Based Servers](class-based-servers.md) — Organize complex servers
- 💡 [Examples](/_examples.md) — Complete working code
