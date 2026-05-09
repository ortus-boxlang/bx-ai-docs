---
icon: globe
---

# HTTP Endpoint & Routing

Set up HTTP endpoints for your MCP server.

## Built-In Endpoint

The module provides a pre-built HTTP endpoint at `public/mcp.bxm`:

```
POST http://localhost/~bxai/mcp.bxm
```

Specify a server using query parameter or URL segment:

```bash
# Query parameter
POST http://localhost/~bxai/mcp.bxm?server=myApp

# URL segment
POST http://localhost/~bxai/mcp.bxm/myApp
```

## Discovery (GET)

Get server capabilities:

```bash
curl http://localhost/~bxai/mcp.bxm?server=myApp
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "protocolVersion": "2024-11-05",
        "capabilities": {
            "tools": {},
            "resources": {},
            "prompts": {}
        },
        "serverInfo": {
            "name": "myApp",
            "version": "1.0.0"
        }
    }
}
```

## Custom Entry Points

Create your own MCP endpoint at any URL:

```javascript
// api/mcp-endpoint.bxm
<bx:script>
import bxModules.bxai.models.mcp.MCPRequestProcessor
MCPRequestProcessor::startHttp()
</bx:script>
```

Now you have a fully functional endpoint at:

```
POST http://localhost/api/mcp-endpoint.bxm?server=myApp
```

## Custom Routing Examples

### Secured Admin Endpoint

```javascript
// admin/mcp-admin.bxm
<bx:script>
// Check admin authentication
if ( !session.isAdmin ) {
    writeOutput( serializeJSON({
        jsonrpc: "2.0",
        error: {
            code: -32000,
            message: "Unauthorized"
        },
        id: null
    }) )
    abort;
}

// Render MCP processor
import bxModules.bxai.models.mcp.MCPRequestProcessor
MCPRequestProcessor::startHttp()
</bx:script>
```

### API Versioned Endpoint

```javascript
// api/v2/ai/mcp.bxm
<bx:script>
// Set API version header
request.setHeader( "X-API-Version", "2.0" )

// Render MCP processor
import bxModules.bxai.models.mcp.MCPRequestProcessor
MCPRequestProcessor::startHttp()
</bx:script>
```

### Multi-Server Router

```javascript
// api/mcp-router.bxm
<bx:script>
import bxModules.bxai.models.mcp.MCPRequestProcessor

// Route based on path
var pathSegments = listToArray( url.currentUrl.path, "/" )
var serverName = pathSegments[ arrayLen( pathSegments ) ] ?: url.server ?: "default"

// Validate server name
var validServers = [ "public", "admin", "internal" ]
if ( !arrayContains( validServers, serverName ) ) {
    writeOutput( serializeJSON({
        jsonrpc: "2.0",
        error: {
            code: -32000,
            message: "Invalid server name"
        },
        id: null
    }) )
    abort;
}

// Set server name in request scope for processor
request.mcpServerName = serverName

// Render MCP processor
MCPRequestProcessor::startHttp()
</bx:script>
```

## MCPRequestProcessor

The `MCPRequestProcessor` handles all HTTP request/response logic:

```javascript
import bxModules.bxai.models.mcp.MCPRequestProcessor

// Process HTTP requests
MCPRequestProcessor::startHttp()
```

**What it does automatically:**
- ✅ Extracts server name from query parameter or URL segment
- ✅ Parses JSON-RPC 2.0 requests
- ✅ Routes to correct MCP server instance
- ✅ Returns formatted JSON-RPC responses
- ✅ Fires all MCP events (onMCPRequest, onMCPResponse, onMCPError)
- ✅ Applies security checks and headers
- ✅ Handles CORS preflight requests

## Static Server Management

Check server existence and manage registry:

```javascript
// Check if a server exists
exists = bxModules.bxai.models.mcp.MCPServer::hasInstance( "myApp" )

// Get all server names
names = bxModules.bxai.models.mcp.MCPServer::getInstanceNames()
// [ "default", "api", "admin" ]

// Remove a server
wasRemoved = bxModules.bxai.models.mcp.MCPServer::removeInstance( "oldApp" )

// Clear all servers
bxModules.bxai.models.mcp.MCPServer::clearAllInstances()
```

## Use Cases

### Custom Routing

```
/api/v1/mcp      — Legacy API endpoint
/api/v2/mcp      — Current API endpoint
/admin/mcp       — Admin tools (authentication required)
/webhooks/mcp    — Webhook-triggered tools
```

### Framework Integration

Integrate MCP servers into existing URL schemes:

```javascript
// framework/routes.bx
routes = [
    { pattern: "/api/mcp/:server", handler: "mcp.handler.processRequest" },
    { pattern: "/admin/mcp", handler: "mcp.handler.adminRequest" }
]
```

### Security

Put endpoints behind middleware:

```javascript
// middleware/mcpAuth.bx
class {
    function run() {
        // Custom auth logic
        if ( !session.isAuthenticated ) {
            abort 403
        }
    }
}
```

## Testing

### With curl

```bash
# List tools
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'

# Call a tool
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "id": "2",
    "params": {
      "name": "search",
      "arguments": { "query": "test" }
    }
  }'
```

### With Postman

1. Create POST request to `http://localhost/~bxai/mcp.bxm?server=myApp`
2. Set `Content-Type: application/json`
3. Use JSON body with MCP methods
4. Send and inspect responses

## Next Steps

- 🔐 [Server Configuration](server-configuration.md) — Auth & security
- 📊 [Observability](observability.md) — Monitor requests
- ✅ [Best Practices](best-practices.md) — Production setup
