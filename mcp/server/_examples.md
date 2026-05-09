---
icon: code
---

# Examples & Use Cases

Complete working examples for common MCP server scenarios.

## Complete Application Setup

```javascript
// Application.bx
class {

    function onApplicationStart() {
        // Create public server
        MCPServer( "public" )
            .setDescription( "Public API Server" )
            .setVersion( "1.0.0" )
            .withCors( "*" )
            .registerTool(
                aiTool( "searchProducts", "Search product catalog", ( query ) => {
                    return productService.search( query )
                } )
            )
            .registerTool(
                aiTool( "getProduct", "Get product details", ( productId ) => {
                    return productService.getById( productId )
                } )
            )

        // Create admin server (authenticated)
        MCPServer( "admin" )
            .setDescription( "Admin Tools" )
            .setVersion( "1.0.0" )
            .withBasicAuth( getEnv( "ADMIN_USER" ), getEnv( "ADMIN_PASS" ) )
            .registerTool(
                aiTool( "createProduct", "Create new product", ( name, price ) => {
                    return productService.create( name, price )
                } )
            )
            .registerTool(
                aiTool( "deleteProduct", "Delete product", ( productId ) => {
                    return productService.delete( productId )
                } )
            )

        return true
    }

    function onApplicationEnd() {
        bxModules.bxai.models.mcp.MCPServer::clearAllInstances()
    }

}
```

## Class-Based Server Example

```javascript
// mcp/ApplicationServer.bx
class extends="MCPServer" {

    function init() {
        super.init(
            name: "application",
            description: "Application MCP Server",
            version: "1.0.0"
        )

        this.withCors( getEnv( "CORS_ORIGINS", "*" ) )
            .withBodyLimit( 1048576 )

        // Auto-discover annotated tools
        this.scan( "models.tools" )

        return this
    }

}

// models/tools/ProductTools.bx
class {

    @mcpTool( "Search products" )
    function searchProducts( required string query ) {
        return productService.search( arguments.query )
    }

    @mcpTool( "Get product" )
    function getProduct( required string productId ) {
        return productService.getById( arguments.productId )
    }

}

// Application.bx
class {

    function onApplicationStart() {
        aiService().putServer( "app", new mcp.ApplicationServer() )
    }

}
```

## Multi-Server Setup with Monitoring

```javascript
// Application.bx
class {

    function onApplicationStart() {
        // Public API
        MCPServer( "public" )
            .registerTool( publicSearchTool )

        // Admin API
        MCPServer( "admin" )
            .withBasicAuth( getEnv( "ADMIN_USER" ), getEnv( "ADMIN_PASS" ) )
            .registerTool( adminTool )

        // Register for monitoring
        BoxRegisterInterceptor( this, "onMCPResponse,onMCPError" )
    }

    function onMCPResponse( event, interceptData ) {
        var server = interceptData.server
        var method = interceptData.requestData.method

        // Send to monitoring
        metrics.gauge( "mcp.responseTime", interceptData.responseTime, {
            tags: [ "server:#server.getServerName()#", "method:#method#" ]
        } )
    }

    function onMCPError( event, interceptData ) {
        var exception = interceptData.exception

        // Alert on errors
        writeLog(
            type: "error",
            file: "mcp-errors",
            text: "Error in #interceptData.context#: #exception.message#"
        )
    }

}
```

## Using MCP Client to Connect

```javascript
// Connect to the public server
client = MCP( "http://localhost/~bxai/mcp.bxm?server=public" )

// List available tools
tools = client.listTools()

// Invoke a tool
result = client.callTool( "searchProducts", { query: "laptop" } )

if ( result.success ) {
    products = result.data.content
    writeOutput( "Found #arrayLen( products )# products" )
}
```

## Custom HTTP Endpoint

```javascript
// api/mcp.bxm
<bx:script>
import bxModules.bxai.models.mcp.MCPRequestProcessor

// Optional: Log all requests
BoxRegisterInterceptor( "onMCPRequest", function( event, interceptData ) {
    writeLog(
        type: "information",
        file: "mcp-api",
        text: "API Request: #interceptData.requestData.method#"
    )
})

// Start HTTP processor
MCPRequestProcessor::startHttp()
</bx:script>
```

## STDIO Transport Entry Point

```javascript
// mcp-stdio.bxs
import bxModules.bxai.models.mcp.MCPRequestProcessor

// Create server
MCPServer( "stdio-server" )
    .registerTool(
        aiTool( "echo", "Echo tool", msg => msg )
    )

// Get server name from environment
var serverName = getSystemSetting( "MCP_SERVER_NAME", "default" )

// Start STDIO transport
MCPRequestProcessor::startStdio( serverName )
```

Run: `boxlang mcp-stdio.bxs --server stdio-server`

## Complete Example: E-Commerce API

```javascript
// Application.bx
class {

    function onApplicationStart() {
        // Public: Read-only catalog operations
        MCPServer( "ecommerce-public" )
            .setDescription( "E-Commerce Public API" )
            .setVersion( "1.0.0" )
            .withCors( "*" )
            .registerTool(
                aiTool( "searchProducts", "Search products", q => productService.search( q ) )
            )
            .registerTool(
                aiTool( "getProduct", "Get product details", id => productService.getById( id ) )
            )
            .registerTool(
                aiTool( "getCategories", "List product categories", () => productService.getCategories() )
            )

        // Internal: Full access (authenticated)
        MCPServer( "ecommerce-internal" )
            .setDescription( "E-Commerce Internal API" )
            .setVersion( "1.0.0" )
            .withBasicAuth( "admin", getEnv( "INTERNAL_PASS" ) )
            .registerTool(
                aiTool( "createOrder", "Create order", ( items, customer ) => orderService.create( items, customer ) )
            )
            .registerTool(
                aiTool( "updateOrder", "Update order status", ( orderId, status ) => orderService.update( orderId, status ) )
            )

        // Register monitoring
        BoxRegisterInterceptor( this, "onMCPResponse" )
    }

    function onMCPResponse( event, interceptData ) {
        var stats = interceptData.server.getStatsSummary()

        // Log performance
        if ( stats.avgResponseTime > 200 ) {
            writeLog(
                type: "warning",
                file: "performance",
                text: "Slow MCP response: #stats.avgResponseTime#ms"
            )
        }
    }

}
```

## Next Steps

- 🚀 [Getting Started](getting-started.md) — Your first server
- 🏗️ [Class-Based Servers](class-based-servers.md) — Organize complex servers
- ✅ [Best Practices](best-practices.md) — Production setup
