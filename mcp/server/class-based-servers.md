---
icon: building
---

# Class-Based MCP Servers

For larger applications, encapsulate your entire MCP server in a dedicated class that extends `MCPServer`. This provides better organization, reusability, and testability.

## Overview

Instead of creating inline servers with `MCPServer( "name" )`, extend the `MCPServer` class and register it with `aiService().putServer()`.

## Basic Class Structure

```javascript
// mcp/MyAppServer.bx
class extends="MCPServer" {

    function init() {
        // Initialize the parent MCPServer
        super.init(
            name: "my-app",
            description: "My Application MCP Server",
            version: "1.0.0"
        )

        // Register capabilities in constructor
        registerTools()
        registerResources()
        registerPrompts()
        configureSecurity()

        return this
    }

    function registerTools() {
        this.registerTool(
            aiTool( "search", "Search documents", query => searchService.search( query ) )
        )
        this.registerTool(
            aiTool( "calculate", "Perform calculations", expr => evaluate( expr ) )
        )
    }

    function registerResources() {
        this.registerResource(
            uri: "docs://api",
            name: "API Documentation",
            description: "API reference",
            mimeType: "text/markdown",
            handler: () => fileRead( expandPath( "/docs/api.md" ) )
        )
    }

    function registerPrompts() {
        this.registerPrompt(
            name: "summarize",
            description: "Summarize a document",
            args: [
                { name: "text", description: "Text to summarize", required: true }
            ],
            handler: ( args ) => [
                { role: "system", content: "You are a summarization assistant." },
                { role: "user", content: "Summarize: #args.text#" }
            ]
        )
    }

    function configureSecurity() {
        this.withBasicAuth( getEnv( "MCP_USER" ), getEnv( "MCP_PASS" ) )
            .withCors( "https://myapp.com" )
    }

}
```

## Register at Application Start

```javascript
// Application.bx
class {

    function onApplicationStart() {
        // Create and register the server instance
        var server = new mcp.MyAppServer()
        aiService().putServer( server.getServerName(), server )

        return true
    }

    function onApplicationEnd() {
        // Clean up on shutdown
        aiService().removeServer( "my-app" )
    }

}
```

## Benefits vs Inline

| Inline `MCPServer()` | Class-Based `extends="MCPServer"` |
|---|---|
| Quick setup for simple servers | Better organization for complex servers |
| All config in one place | Separation of concerns |
| Harder to test in isolation | Easily unit-testable |
| Good for scripts & prototypes | Good for production applications |
| | Supports inheritance & composition |

## Multiple Server Classes

Create different server classes for different purposes:

```javascript
// mcp/PublicServer.bx
class extends="MCPServer" {
    function init() {
        super.init( name: "public", description: "Public API", version: "1.0.0" )
        this.scan( "models.public" )  // Auto-discover public tools
        return this
    }
}

// mcp/AdminServer.bx
class extends="MCPServer" {
    function init() {
        super.init( name: "admin", description: "Admin Tools", version: "1.0.0" )
        this.scan( "models.admin" )  // Auto-discover admin tools
        this.withBasicAuth( getEnv( "ADMIN_USER" ), getEnv( "ADMIN_PASS" ) )
        return this
    }
}

// Application.bx
function onApplicationStart() {
    aiService().putServer( "public", new mcp.PublicServer() )
    aiService().putServer( "admin", new mcp.AdminServer() )
}
```

## Using Annotation Discovery in Classes

Combine class structure with automatic discovery:

```javascript
// mcp/DiscoveryServer.bx
class extends="MCPServer" {

    function init() {
        super.init(
            name: "discovery-server",
            description: "Server with auto-discovered tools",
            version: "1.0.0"
        )

        // Auto-discover all annotated methods in a package
        this.scan( "models.tools" )

        // Or scan a specific class
        this.scanClass( new models.special.SpecialTools() )

        return this
    }

}
```

## Inheritance & Composition

### Base Server Class

```javascript
// mcp/BaseAppServer.bx
class extends="MCPServer" abstract {

    function init( required string name ) {
        super.init(
            name: arguments.name,
            version: getSystemSetting( "APP_VERSION" ),
            description: getSystemSetting( "APP_DESCRIPTION" )
        )

        // Common configuration
        this.withCors( getEnv( "CORS_ORIGINS", "*" ) )

        // Let subclasses register specific tools
        registerTools()

        return this
    }

    function registerTools() {
        // Abstract - override in subclasses
    }

}
```

### Specialized Server

```javascript
// mcp/SearchServer.bx
class extends="mcp.BaseAppServer" {

    function init() {
        super.init( "search" )
        return this
    }

    function registerTools() {
        this.registerTool(
            aiTool( "search", "Search", args => searchService.search( args.query ) )
        )
    }

}
```

## Configuration via Constructor Parameters

```javascript
// mcp/ConfigurableServer.bx
class extends="MCPServer" {

    property name="config" type="struct" default="{}";

    function init( required struct config = {} ) {
        this.config = arguments.config

        super.init(
            name: this.config.name ?: "default",
            description: this.config.description ?: "",
            version: this.config.version ?: "1.0.0"
        )

        // Apply dynamic configuration
        if ( this.config.keyExists( "requireAuth" ) && this.config.requireAuth ) {
            this.withBasicAuth(
                this.config.authUser ?: "admin",
                this.config.authPass ?: ""
            )
        }

        if ( this.config.keyExists( "corsOrigins" ) ) {
            this.withCors( this.config.corsOrigins )
        }

        registerTools()
        return this
    }

    function registerTools() {
        // Subclasses override this
    }

}
```

### Using Configurable Server

```javascript
// Application.bx
function onApplicationStart() {
    aiService().putServer( "api", new mcp.ConfigurableServer( {
        name: "api",
        description: "Public API Server",
        version: "2.0.0",
        corsOrigins: "*"
    } ) )

    aiService().putServer( "admin", new mcp.ConfigurableServer( {
        name: "admin",
        description: "Admin Server",
        requireAuth: true,
        authUser: getEnv( "ADMIN_USER" ),
        authPass: getEnv( "ADMIN_PASS" ),
        corsOrigins: "https://admin.example.com"
    } ) )
}
```

## Unit Testing Class-Based Servers

```javascript
// test/MyAppServerTest.bx
class extends="org.testbox.system.BaseSpec" {

    function run( testResults, testBox ) {
        describe( "MyAppServer", () => {

            it( "should initialize with correct name", () => {
                var server = new mcp.MyAppServer()
                expect( server.getServerName() ).toBe( "my-app" )
            } )

            it( "should have search tool registered", () => {
                var server = new mcp.MyAppServer()
                expect( server.hasTool( "search" ) ).toBeTrue()
            } )

            it( "should process tool calls", () => {
                var server = new mcp.MyAppServer()
                var result = server.handleRequest( {
                    jsonrpc: "2.0",
                    method: "tools/call",
                    id: "1",
                    params: {
                        name: "search",
                        arguments: { query: "test" }
                    }
                } )

                expect( result.jsonrpc ).toBe( "2.0" )
                expect( result.keyExists( "result" ) ).toBeTrue()
            } )

        } )
    }

}
```

## Next Steps

- 🧩 [Registering Tools](registration.md) — Manual registration
- 📋 [Annotation Discovery](annotation-discovery.md) — Auto-register
- 🔐 [Server Configuration](server-configuration.md) — Security & auth
- 💡 [Examples](/_examples.md) — Complete working projects
