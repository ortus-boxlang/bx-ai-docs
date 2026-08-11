---
description: Expose your application's tools, resources, and prompts to AI clients through a complete MCP server implementation.
icon: server
---

# MCP Server - Model Context Protocol Server

The BoxLang AI Module provides a complete MCP (Model Context Protocol) server implementation that allows you to expose tools, resources, and prompts to AI clients.

## What is an MCP Server?

An MCP Server is a service that exposes capabilities to AI clients using the standardized Model Context Protocol. It enables:

* 🔧 **Expose Tools**: Register functions that AI clients can invoke
* 📚 **Serve Resources**: Provide documents and data to AI clients
* 💬 **Offer Prompts**: Define reusable prompt templates
* 🌐 **HTTP & STDIO Transports**: Expose your MCP server via web or command-line

## 🏗️ MCP Architecture

```mermaid
graph TB
    subgraph "AI Clients"
        C1[Claude Desktop]
        C2[VS Code]
        C3[Web App]
        C4[Custom Client]
    end

    subgraph "Transport Layer"
        H[HTTP Transport]
        S[STDIO Transport]
    end

    subgraph "MCP Server"
        MS[MCP Server Core]
        TR[Tool Registry]
        RR[Resource Registry]
        PR[Prompt Registry]
    end

    subgraph "Your Application"
        T[Tools/Functions]
        D[Data Sources]
        P[Prompt Templates]
    end

    C1 --> S
    C2 --> S
    C3 --> H
    C4 --> H

    H --> MS
    S --> MS

    MS --> TR
    MS --> RR
    MS --> PR

    TR --> T
    RR --> D
    PR --> P

    style MS fill:#BD10E0
    style H fill:#4A90E2
    style S fill:#7ED321
    style TR fill:#B8E986
```

## Documentation

| Topic | Description |
|---|---|
| 🚀 [Getting Started](getting-started.md) | Build your first server in 5 minutes |
| 🌐 [Transports](transports.md) | HTTP vs STDIO — choose what fits your deployment |
| 🔧 [Server Configuration](server-configuration.md) | Authentication, CORS, IP allow lists, body limits, API keys |
| 🧩 [Registering Tools, Resources & Prompts](registration.md) | Manual registration with inline MCPServer calls |
| ✨ [Annotation-Based Discovery](annotation-discovery.md) | Auto-register with @mcpTool, @mcpResource, @mcpPrompt |
| 🏗️ [Class-Based Servers](class-based-servers.md) | Extend MCPServer for better organization |
| 🌍 [HTTP Endpoint & Routing](http-endpoint.md) | Set up custom HTTP endpoints |
| 📊 [Observability & Monitoring](observability.md) | Statistics, events, and real-time monitoring |
| ⏸️ [Pause & Resume](pause-resume.md) | Temporarily halt servers for maintenance |
| ⭐ [Best Practices](best-practices.md) | Security, design patterns, production setup |
| 💡 [Examples & Use Cases](_examples.md) | Complete working code examples |

## Quick Start

```javascript
// Application.bx
class {

    function onApplicationStart() {
        MCPServer( "myApp" )
            .setDescription( "My Application MCP Server" )
            .setVersion( "1.0.0" )
            .registerTool(
                aiTool( "search", "Search documents", ( query ) => {
                    return searchService.search( query )
                } )
            )
    }

}
```

Access at: `POST http://localhost/~bxai/mcp.bxm?server=myApp`

👉 **[Full Getting Started Guide →](getting-started.md)**

## External Resources

* [Model Context Protocol Specification](https://modelcontextprotocol.io)
* [MCP Implementation Examples](https://github.com/modelcontextprotocol)
