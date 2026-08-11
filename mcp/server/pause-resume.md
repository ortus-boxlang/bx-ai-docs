---
description: Temporarily halt an MCP server without destroying its configuration.
icon: pause-circle
---

# Pause & Resume

Temporarily halt an MCP server without destroying its configuration.

{% hint style="info" %}
**Since BoxLang AI v3.2.0+**
{% endhint %}

## Overview

Pausing a server keeps it registered but rejects all incoming requests (except `ping`) with a `SERVER_PAUSED` error (code `-32005`).

**Useful for:**
- Maintenance windows
- Admin interfaces that need to disable a server temporarily
- Rate limiting or circuit breaker patterns
- Graceful shutdown sequences

## Basic Usage

```javascript
server = MCPServer( "my-tools" )
    .registerTool( searchTool )
    .registerTool( createTicketTool )

// Pause the server
server.pause()

// Check if paused
if ( server.isPaused() ) {
    println( "Server is currently paused" )
}

// Resume the server
server.resume()
```

## Fluent Chaining

```javascript
server = MCPServer( "admin-tools" )
    .registerTool( adminTool )
    .pause()      // Start paused
    .resume()     // Resume when ready
```

## Events

Pausing and resuming fire interception points:

| Event | When |
|---|---|
| `onMCPServerPause` | When `pause()` called |
| `onMCPServerResume` | When `resume()` called |

```javascript
BoxRegisterInterceptor( "onMCPServerPause", function( event ) {
    println( "MCP server paused: #event.name#" )
    // Notify admin, send alert, etc.
})

BoxRegisterInterceptor( "onMCPServerResume", function( event ) {
    println( "MCP server resumed: #event.name#" )
})
```

## Error Response When Paused

When a paused server receives a request (except `ping`), it returns:

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "error": {
        "code": -32005,
        "message": "Server is paused"
    }
}
```

## Monitoring Pause State

```javascript
summary = server.getStatsSummary()
println( "Paused: #summary.paused#" )  // true or false
```

## Example: Maintenance Mode

```javascript
// Application.bx
class {

    function onApplicationStart() {
        MCPServer( "myApp" )
            .registerTool( productTool )
            .registerTool( orderTool )
    }

    function enterMaintenance() {
        var server = MCPServer( "myApp" )
        if ( !isNull( server ) ) {
            server.pause()
            writeLog( "Server paused for maintenance" )
        }
    }

    function exitMaintenance() {
        var server = MCPServer( "myApp" )
        if ( !isNull( server ) ) {
            server.resume()
            writeLog( "Server resumed" )
        }
    }

}
```

## Next Steps

- ✅ [Best Practices](best-practices.md) — Production patterns
- 📊 [Observability](observability.md) — Monitor operations
- 💡 [Examples](_examples.md) — Complete examples
