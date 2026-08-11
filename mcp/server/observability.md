---
description: Track performance, monitor requests, and respond to MCP server events in real-time.
icon: chart-line
---

# Observability & Monitoring

Track performance, monitor requests, and respond to events in real-time.

## Statistics & Monitoring

The MCP server automatically tracks metrics for monitoring and analytics.

### Enabling Statistics

Stats are **enabled by default**:

```javascript
// Stats enabled by default
server = MCPServer( "myApp" )

// Explicitly disable
server = MCPServer( name: "myApp", statsEnabled: false )
```

### Get Summary Statistics

Quick overview of key metrics:

```javascript
summary = server.getStatsSummary()

writeOutput( "Total Requests: #summary.totalRequests#" )
writeOutput( "Success Rate: #summary.successRate#%" )
writeOutput( "Avg Response Time: #summary.avgResponseTime#ms" )
writeOutput( "Tool Calls: #summary.totalToolInvocations#" )
writeOutput( "Resource Reads: #summary.totalResourceReads#" )
writeOutput( "Errors: #summary.totalErrors#" )
writeOutput( "Uptime: #summary.uptime / 1000#s" )
```

**Fields:**
- `uptime` — Server uptime in milliseconds
- `totalRequests` — Total requests processed
- `successRate` — Success rate as percentage
- `avgResponseTime` — Average response time in ms
- `requestsPerMinute` — Request rate
- `activeRequests` — Currently processing requests
- `paused` — Whether server is paused
- `totalToolInvocations` — Total tool calls
- `totalResourceReads` — Total resource reads
- `totalPromptGenerations` — Total prompt generations
- `totalErrors` — Total errors encountered
- `security.authFailures` — Failed HTTP basic auth attempts
- `security.apiKeyFailures` — Failed API key checks
- `security.bodySizeViolations` — Rejected oversized payloads
- `security.ipFilterFailures` — Rejected client IP addresses

### Get Detailed Statistics

Complete stats with breakdowns:

```javascript
stats = server.getStats()

// Request breakdown
writeOutput( "Successful: #stats.requests.successful#" )
writeOutput( "Failed: #stats.requests.failed#" )
writeOutput( "By Method: #serializeJSON( stats.requests.byMethod )#" )

// Tool breakdown
writeOutput( "Tools by name: #serializeJSON( stats.tools.byTool )#" )
writeOutput( "Avg Execution Time: #stats.tools.avgExecutionTime#ms" )

// Error breakdown
writeOutput( "Errors by code: #serializeJSON( stats.errors.byCode )#" )
if ( !stats.errors.lastError.isEmpty() ) {
    writeOutput( "Last Error: #stats.errors.lastError.message#" )
}

// Security breakdown
writeOutput( "Auth failures: #stats.security.authFailures#" )
writeOutput( "API key failures: #stats.security.apiKeyFailures#" )
writeOutput( "Body size violations: #stats.security.bodySizeViolations#" )
writeOutput( "IP filter failures: #stats.security.ipFilterFailures#" )
```

### Managing Statistics

```javascript
// Check if enabled
if ( server.isStatsEnabled() ) {
    writeOutput( "Stats tracking is active" )
}

// Disable tracking
server.disableStats()

// Re-enable tracking
server.enableStats()

// Reset all statistics
server.resetStats()
```

## Event System 🎯

The MCP server fires custom events during its lifecycle, allowing you to add logging, monitoring, and custom integration.

### Available Events

#### `onMCPServerCreate`

Fired when an MCP server instance is created.

```javascript
function onMCPServerCreate( event, interceptData ) {
    writeLog(
        type: "information",
        file: "mcp",
        text: "MCP server created: #interceptData.name#"
    )
}
```

#### `onMCPServerRemove`

Fired when an MCP server is removed.

```javascript
function onMCPServerRemove( event, interceptData ) {
    writeLog(
        type: "information",
        file: "mcp",
        text: "MCP server removed: #interceptData.name#"
    )
}
```

#### `onMCPRequest`

Fired before processing an incoming MCP request.

```javascript
function onMCPRequest( event, interceptData ) {
    // interceptData contains:
    // - server: The MCPServer instance
    // - requestData: { id, method, params }

    writeLog(
        type: "information",
        file: "mcp-requests",
        text: "Request: #interceptData.requestData.method# (ID: #interceptData.requestData.id#)"
    )
}
```

#### `onMCPResponse`

Fired after processing and before returning the response.

```javascript
function onMCPResponse( event, interceptData ) {
    // interceptData contains:
    // - server: The MCPServer instance
    // - requestData: The original request
    // - responseData: The response
    // - success: Whether successful
    // - responseTime: Time in milliseconds

    if ( !interceptData.success ) {
        writeLog(
            type: "warning",
            file: "mcp-failures",
            text: "Failed request: #interceptData.requestData.method#"
        )
    }
}
```

#### `onMCPError`

Fired when an exception occurs.

```javascript
function onMCPError( event, interceptData ) {
    // interceptData contains:
    // - server: The MCPServer instance
    // - context: Where error occurred
    // - exception: The exception object
    // - Additional context-specific fields

    var exception = interceptData.exception

    writeLog(
        type: "error",
        file: "mcp-errors",
        text: "MCP Error: #exception.message#"
    )
}
```

### Registering Event Listeners

#### Module Registration

In `ModuleConfig.bx`:

```javascript
function configure() {
    interceptors = [
        { class: "MyMCPInterceptor" }
    ];
}
```

#### Application Registration

Use `BoxRegisterInterceptor()`:

```javascript
// Application.bx
class {

    function onApplicationStart() {
        // Register for MCP events
        BoxRegisterInterceptor( this, "onMCPRequest,onMCPResponse,onMCPError" )

        MCPServer( "myApp" )
            .registerTool( myTool )
    }

    function onMCPRequest( event, interceptData ) {
        metrics.increment( "mcp.requests" )
    }

    function onMCPResponse( event, interceptData ) {
        metrics.gauge( "mcp.responseTime", interceptData.responseTime )
    }

    function onMCPError( event, interceptData ) {
        errorTracker.captureException( interceptData.exception )
    }

}
```

## Practical Examples

### Real-Time Dashboard

```javascript
writeOutput( "
    <div class='stats-dashboard'>
        <h2>MCP Server: myApp</h2>

        <div class='stat-box'>
            <label>Uptime</label>
            <value>#numberFormat( stats.uptime / 1000, '0' )#s</value>
        </div>

        <div class='stat-box'>
            <label>Total Requests</label>
            <value>#stats.totalRequests#</value>
        </div>

        <div class='stat-box'>
            <label>Success Rate</label>
            <value>#numberFormat( stats.successRate, '0.00' )#%</value>
        </div>

        <div class='stat-box'>
            <label>Avg Response</label>
            <value>#numberFormat( stats.avgResponseTime, '0' )#ms</value>
        </div>

        <div class='stat-box error'>
            <label>Errors</label>
            <value>#stats.totalErrors#</value>
        </div>
    </div>
" )
```

### Custom Metrics

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

        // Alert on high error rate
        if ( summary.successRate < 90 && summary.totalRequests > 10 ) {
            writeLog(
                type: "error",
                file: "mcp-alerts",
                text: "High error rate: #summary.successRate#%"
            )
        }

        // Alert on slow responses
        if ( summary.avgResponseTime > 1000 ) {
            writeLog(
                type: "warning",
                file: "mcp-alerts",
                text: "Slow avg response: #summary.avgResponseTime#ms"
            )
        }
    }

}
```

### Integration with Monitoring Services

```javascript
function onMCPResponse( event, interceptData ) {
    // Send to DataDog
    datadog.metric( "mcp.responseTime", interceptData.responseTime, {
        tags: [ "server:#interceptData.server.getServerName()#" ]
    })

    // Send to Prometheus
    prometheus.observe(
        metric: "mcp_response_time_seconds",
        value: interceptData.responseTime / 1000,
        labels: { server: interceptData.server.getServerName() }
    )
}
```

### REST API Endpoint

Expose stats via API:

```javascript
class {

    function getServerStats( required string serverName ) {
        var server = MCPServer( arguments.serverName )

        if ( isNull( server ) ) {
            return { success: false, error: "Server not found" }
        }

        return {
            success: true,
            data: server.getStatsSummary()
        }
    }

    function resetServerStats( required string serverName ) {
        var server = MCPServer( arguments.serverName )

        if ( isNull( server ) ) {
            return { success: false }
        }

        server.resetStats()

        return {
            success: true,
            message: "Statistics reset successfully"
        }
    }

}
```

## Performance Notes

- **Zero Overhead When Disabled** — No impact if `statsEnabled: false`
- **Memory Efficient** — Last 1000 samples retained per metric
- **Thread Safe** — Uses atomic operations
- **Real-Time** — Stats updated immediately
- **Lightweight Summary** — `getStatsSummary()` optimized for frequent polling

## Next Steps

- ⏸️ [Pause & Resume](pause-resume.md) — Pause server operations
- ✅ [Best Practices](best-practices.md) — Production monitoring
- 💡 [Examples](_examples.md) — Complete working setups
