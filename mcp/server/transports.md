---
description: Choose between HTTP and STDIO transport mechanisms for your MCP server based on deployment scenario.
icon: network-wired
---

# Transport Types: HTTP vs STDIO

MCP servers can communicate via two transport mechanisms. Choose based on your deployment scenario.

## 🌐 HTTP Transport (Web)

**Best for:** Web applications, REST APIs, browser-based clients, any HTTP-capable client

### How It Works

Clients send JSON-RPC requests over HTTP POST to a `.bxm` endpoint. The server processes and returns JSON-RPC responses.

### Built-In Endpoint

Use the automatic endpoint provided by the module:

```
POST http://localhost/~bxai/mcp.bxm?server=myApp
```

Or create a custom endpoint:

```javascript
// api/my-mcp.bxm
<bx:script>
import bxModules.bxai.models.mcp.MCPRequestProcessor
MCPRequestProcessor::startHttp()
</bx:script>
```

### Features

✅ CORS support with wildcard patterns
✅ Body size limits
✅ API key authentication
✅ HTTP Basic Auth
✅ Security headers
✅ Server-Sent Events (SSE) streaming
✅ Discovery endpoint (GET)

### Making Requests

```bash
# List tools
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'

# Discovery (GET)
curl http://localhost/~bxai/mcp.bxm?server=myApp
```

## 🖥️ STDIO Transport (Command-Line)

**Best for:** Desktop applications, CLI tools, IDE integrations, local AI assistants (Claude Desktop, etc.)

### How It Works

Clients communicate via standard input/output (STDIN/STDOUT). Requests and responses are JSON-RPC formatted, one per line.

### Create a Script Entry Point

```javascript
// mcp-stdio.bxs
import bxModules.bxai.models.mcp.MCPRequestProcessor

// Register your server and tools
MCPServer( "myApp" )
    .registerTool( aiTool( "echo", "Echo tool", ( msg ) => msg ) )

// Get server name from CLI args
serverName = getSystemSetting( "MCP_SERVER_NAME", "default" )
parsedArgs = cliGetArgs()

if ( structKeyExists( parsedArgs.options, "server" ) ) {
    serverName = parsedArgs.options.server
}

// Start STDIO transport (blocks until shutdown)
MCPRequestProcessor::startStdio( serverName )
```

### Run STDIO Server

```bash
# Start with default server
boxlang mcp-stdio.bxs

# Start with specific server
boxlang mcp-stdio.bxs --server myApp

# Or via environment variable
MCP_SERVER_NAME=myApp boxlang mcp-stdio.bxs
```

### Features

✅ JSON-RPC over STDIN/STDOUT
✅ Line-based protocol
✅ Graceful shutdown signal
✅ Process lifecycle management
❌ No HTTP headers/CORS (not needed)
❌ No status codes (JSON-RPC error codes used)

### STDIO Communication

```bash
# Input (STDIN)
{"jsonrpc":"2.0","method":"tools/list","params":{},"id":1}

# Output (STDOUT)
{"jsonrpc":"2.0","result":{"tools":[...]},"id":1}

# Shutdown
{"jsonrpc":"2.0","method":"shutdown","params":{},"id":99}
```

## 🎯 Choosing a Transport

| Scenario | Recommended |
|---|---|
| Web application with browser clients | HTTP |
| REST API for external services | HTTP |
| Desktop AI assistant (Claude Desktop) | STDIO |
| VS Code extension / IDE integration | STDIO |
| Command-line tool | STDIO |
| Docker container as MCP service | Both |
| Testing/Development | HTTP (easier to test with curl) |

## Comparison

| Feature | HTTP | STDIO |
|---|---|---|
| Setup | Automatic or simple `.bxm` | Script entry point |
| Security | Auth headers, CORS, HTTPS | Process isolation |
| Performance | Network latency | Local process |
| Debugging | curl/Postman friendly | Process logs |
| Scalability | Multi-client, reverse proxy | Single client per process |
| Integration | Browser, mobile, web apps | Desktop apps, CLIs, IDEs |

## Examples

### HTTP Example (Claude Desktop)

```json
{
  "mcpServers": {
    "myapp": {
      "url": "http://localhost:8000/~bxai/mcp.bxm?server=myApp",
      "env": {}
    }
  }
}
```

### STDIO Example (Claude Desktop)

```json
{
  "mcpServers": {
    "myapp": {
      "command": "boxlang",
      "args": ["mcp-stdio.bxs", "--server", "myApp"],
      "env": {
        "MCP_SERVER_NAME": "myApp"
      }
    }
  }
}
```

## Next Steps

- 🚀 [Getting Started](getting-started.md) — Your first server
- 🔐 [Server Configuration](server-configuration.md) — Authentication & security
- 🛠️ [HTTP Endpoint](http-endpoint.md) — Custom HTTP routing
