---
description: Guidelines for production MCP servers — security, monitoring, performance, and deployment.
icon: star
---

# Best Practices

Guidelines for production MCP servers.

## Security Checklist 🔒

**Authentication & Authorization**
- ✅ Use `.withBasicAuth()` for sensitive operations
- ✅ Use `.withApiKeyProvider()` for programmatic access
- ✅ Implement role-based access control in auth providers
- ✅ Store credentials in environment variables, not code
- ✅ Rotate credentials periodically

**Network & Transport**
- ✅ Always use HTTPS in production
- ✅ Configure CORS with specific origins (avoid `*`)
- ✅ Use `.withAllowedIPs()` to restrict source networks for sensitive servers
- ✅ Set appropriate request body size limits
- ✅ Monitor for authentication failures

**Server Management**
- ✅ Separate public and admin servers
- ✅ Use different API keys per server
- ✅ Log all authentication failures
- ✅ Implement rate limiting if needed
- ✅ Enable statistics for monitoring

## Design Patterns

### Multi-Server Architecture

```javascript
// Application.bx
function onApplicationStart() {
    // Public API - read-only, no auth
    MCPServer( "public" )
        .registerTool( searchTool )
        .registerTool( getProductTool )
        .withCors( "*" )

    // Internal API - authenticated, full access
    MCPServer( "internal" )
        .withAllowedIPs( [ "10.0.0.0/8", "192.168.0.0/16" ] )
        .withBasicAuth( getEnv( "INTERNAL_USER" ), getEnv( "INTERNAL_PASS" ) )
        .registerTool( allTools )

    // Admin API - encrypted, restricted
    MCPServer( "admin" )
        .withAllowedIPs( [ "10.0.1.0/24", "203.0.113.50" ] )
        .withBasicAuth( getEnv( "ADMIN_USER" ), getEnv( "ADMIN_PASS" ) )
        .withBodyLimit( 10 * 1024 * 1024 )
        .registerTool( systemTool )
}
```

### Progressive Security

```javascript
// Tier 1: Completely open (for testing)
MCPServer( "dev" )
    .registerTool( devTools )

// Tier 2: CORS restricted
MCPServer( "staging" )
    .withCors( "*.staging.example.com" )
    .registerTool( stagingTools )

// Tier 3: Full security
MCPServer( "production" )
    .withAllowedIPs( [ "203.0.113.0/24" ] )
    .withBasicAuth( getEnv( "API_USER" ), getEnv( "API_PASS" ) )
    .withCors( "https://app.example.com" )
    .withBodyLimit( 1048576 )
    .registerTool( productionTools )
```

### Annotation-Based Auto-Discovery

```javascript
// Models organized by scope
models/tools/
├── public/
│   └── PublicTools.bx        // @mcpTool annotated
├── admin/
│   └── AdminTools.bx        // @mcpTool annotated
└── shared/
    └── SharedTools.bx       // @mcpTool annotated

// Application.bx
MCPServer( "public" ).scan( "models.tools.public" )
MCPServer( "admin" ).scan( "models.tools.admin" )
MCPServer( "admin" ).scan( "models.tools.shared" )
```

## Monitoring & Observability

### Health Check Endpoint

```javascript
// api/health.bxm
<bx:script>
var servers = [ "public", "internal", "admin" ]
var status = {}

for ( var serverName in servers ) {
    var server = MCPServer( serverName )
    if ( !isNull( server ) ) {
        status[ serverName ] = {
            paused: server.isPaused(),
            stats: server.getStatsSummary()
        }
    }
}

writeOutput( serializeJSON( status ) )
</bx:script>
```

### Performance Monitoring

```javascript
class {

    function onApplicationStart() {
        MCPServer( "myApp" )
            .registerTool( myTool )

        BoxRegisterInterceptor( this, "onMCPResponse" )
    }

    function onMCPResponse( event, interceptData ) {
        var server = interceptData.server
        var summary = server.getStatsSummary()

        // Alert on performance degradation
        if ( summary.avgResponseTime > 500 ) {
            slackService.sendAlert(
                channel: "##operations",
                text: "MCP server #server.getServerName()# slow: #summary.avgResponseTime#ms"
            )
        }

        // Alert on high error rate
        if ( summary.successRate < 95 && summary.totalRequests > 100 ) {
            slackService.sendAlert(
                channel: "##operations",
                text: "MCP server #server.getServerName()# errors: #100-summary.successRate#%"
            )
        }
    }

}
```

### Detailed Logging

```javascript
function onMCPRequest( event, interceptData ) {
    var data = interceptData.requestData

    writeLog(
        type: "information",
        file: "mcp-audit",
        text: "#data.method# - Client: #getClientIP()# - User: #session.user.id ?: 'anonymous'#"
    )
}

function onMCPError( event, interceptData ) {
    var exception = interceptData.exception

    writeLog(
        type: "error",
        file: "mcp-errors",
        text: "#exception.message# - Context: #interceptData.context# - Stack: #exception.stackTrace#"
    )
}
```

## Documentation

### Tool Descriptions

```javascript
// Good - descriptive
aiTool(
    "searchProducts",
    "Search the product catalog by name, category, or price range",
    handler
)

// Bad - vague
aiTool( "search", "search", handler )
```

### Tool Arguments

```javascript
// Good - clear argument documentation
aiTool( "calculateShipping", "Calculate shipping cost", handler )
    .describeArg( "weight", "Package weight in pounds" )
    .describeArg( "destination", "Destination zip code" )
    .describeArg( "expedited", "Whether to use expedited shipping (true/false)" )

// Tool arguments appear in client discovery
```

## Testing

### Unit Tests

```javascript
class extends="org.testbox.system.BaseSpec" {

    function run( testResults, testBox ) {
        describe( "MCP Server", () => {

            it( "should list tools", () => {
                var server = MCPServer( "test" )
                server.registerTool( testTool )
                var tools = server.listTools()
                expect( arrayLen( tools ) ).toBe( 1 )
            } )

            it( "should handle tool calls", () => {
                var server = MCPServer( "test" )
                server.registerTool( testTool )

                var response = server.handleRequest( {
                    jsonrpc: "2.0",
                    method: "tools/call",
                    id: "1",
                    params: { name: "testTool", arguments: {} }
                } )

                expect( response.keyExists( "result" ) ).toBeTrue()
            } )

        } )
    }

}
```

### Integration Tests

```bash
# Test HTTP endpoint
curl -X POST http://localhost/~bxai/mcp.bxm?server=test \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'

# Test authentication
curl -X POST http://localhost/~bxai/mcp.bxm?server=secure \
  -H "Authorization: Basic invalid" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'
# Should return 401
```

## Performance Optimization

### Server Sizing

```javascript
// Heavy workload - increase body limit and add monitoring
MCPServer( "heavy" )
    .withBodyLimit( 10 * 1024 * 1024 )  // 10MB
```

### Caching

```javascript
// Cache expensive operations
function searchProductsWithCache( query ) {
    var cacheKey = "search_" & query
    var cached = cacheGet( cacheKey )

    if ( !isNull( cached ) ) {
        return cached
    }

    var result = expensiveSearch( query )
    cachePut( cacheKey, result, 3600 )  // 1 hour

    return result
}
```

### Tool Timeouts

```javascript
// Wrap tools with timeout protection
function safeToolCall( tool, args, timeoutMs = 5000 ) {
    var future = asyncRun( () => {
        return tool.invoke( args )
    }, "io-tasks" )

    try {
        return future.get( timeoutMs, java.util.concurrent.TimeUnit.MILLISECONDS )
    } catch ( java.util.concurrent.TimeoutException e ) {
        throw( "Tool execution timeout", "TimeoutError" )
    }
}
```

## Deployment

### Environment Variables

```bash
# .env
MCP_USER=admin
MCP_PASS=secure-password-here
MCP_CORS_ORIGINS=https://app.example.com
MCP_BODY_LIMIT=1048576
```

### Docker

```dockerfile
FROM boxlang:latest

COPY . /app
WORKDIR /app

ENV MCP_USER=${MCP_USER}
ENV MCP_PASS=${MCP_PASS}

CMD ["boxlang", "Application.bx"]
```

### Health Checks

```bash
# Kubernetes liveness probe
curl -f http://localhost:8080/~bxai/mcp.bxm?server=default \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"ping","id":"1"}' \
  || exit 1
```

## Next Steps

- 📊 [Observability](observability.md) — Monitor in production
- 💡 [Examples](_examples.md) — Complete working setups
