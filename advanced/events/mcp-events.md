---
description: >-
  Events fired by MCP (Model Context Protocol) servers and clients — creation,
  removal, pause/resume, requests, responses, and errors.
icon: server
---

# MCP Events

## Server Events

### onMCPServerCreate

Fired when a new MCP (Model Context Protocol) server instance is created.

**When**: MCP server instantiation via `MCPServer()` **Frequency**: Once per unique server creation

#### Event Arguments

| Argument      | Type        | Description                 |
| ------------- | ----------- | ---------------------------- |
| `server`      | `MCPServer` | The created server instance |
| `name`        | `String`    | Server name/identifier      |
| `description` | `String`    | Server description          |
| `version`     | `String`    | Server version              |

#### Example

```java
function onMCPServerCreate( event, interceptData ) {
    var server = interceptData.server;
    var name = interceptData.name;

    // Log server creation
    writeLog(
        text: "MCP Server created: #name# (v#interceptData.version#)",
        type: "info"
    );

    // Apply default configuration
    server.setCors( "*" );

    // Track server registry
    trackMCPServer( name, {
        createdAt: now(),
        description: interceptData.description
    });
}
```

***

### onMCPServerRemove

Fired when an MCP server instance is being removed from the registry.

**When**: Before server removal via `MCPServer::removeInstance()` **Frequency**: Once per server removal

#### Event Arguments

| Argument | Type     | Description                      |
| -------- | -------- | ---------------------------------- |
| `name`   | `String` | Name of the server being removed |

#### Example

```java
function onMCPServerRemove( event, interceptData ) {
    var serverName = interceptData.name;

    // Clean up server resources
    cleanupServerResources( serverName );

    // Log removal
    writeLog(
        text: "MCP Server removed: #serverName#",
        type: "info"
    );

    // Notify connected clients
    notifyClientsOfServerShutdown( serverName );
}
```

***

### onMCPServerPause

Fired when an MCP server is paused.

**When**: On `MCPServer.pause()`

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `server` | `MCPServer` | The MCP server instance |
| `name` | string | Server name |

#### Example

```javascript
BoxRegisterInterceptor( "onMCPServerPause", function( event ) {
    println( "MCP server paused: #event.name#" )
})
```

***

### onMCPServerResume

Fired when an MCP server is resumed.

**When**: On `MCPServer.resume()`

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `server` | `MCPServer` | The MCP server instance |
| `name` | string | Server name |

#### Example

```javascript
BoxRegisterInterceptor( "onMCPServerResume", function( event ) {
    println( "MCP server resumed: #event.name#" )
})
```

***

### onMCPRequest

Fired before processing an incoming MCP request (JSON-RPC 2.0).

**When**: After CORS handling, before request processing **Frequency**: Every MCP request

#### Event Arguments

| Argument      | Type        | Description                                |
| ------------- | ----------- | -------------------------------------------- |
| `server`      | `MCPServer` | The target server instance                 |
| `requestData` | `Struct`    | Request metadata (method, body, urlParams) |
| `serverName`  | `String`    | Server identifier                          |

#### Example

```java
function onMCPRequest( event, interceptData ) {
    var server = interceptData.server;
    var requestData = interceptData.requestData;
    var serverName = interceptData.serverName;

    // Authentication for MCP requests
    if ( !isAuthorized( requestData ) ) {
        throw(
            type: "MCPAuthError",
            message: "Unauthorized MCP request"
        );
    }

    // Rate limiting
    if ( exceedsRateLimit( serverName ) ) {
        throw(
            type: "RateLimitExceeded",
            message: "Too many MCP requests"
        );
    }

    // Log request
    logMCPRequest({
        server: serverName,
        method: requestData.method,
        timestamp: now()
    });
}
```

***

### onMCPResponse

Fired after processing an MCP response, before returning to client.

**When**: After request handling, before HTTP response **Frequency**: Every MCP response

#### Event Arguments

| Argument      | Type        | Description                                               |
| ------------- | ----------- | ------------------------------------------------------------- |
| `server`      | `MCPServer` | The server instance                                       |
| `response`    | `Struct`    | Response data (content, contentType, headers, statusCode) |
| `requestData` | `Struct`    | Original request metadata                                 |
| `serverName`  | `String`    | Server identifier                                          |

#### Example

```java
function onMCPResponse( event, interceptData ) {
    var response = interceptData.response;
    var requestData = interceptData.requestData;
    var serverName = interceptData.serverName;

    // Add custom headers
    response.headers[ "X-MCP-Server" ] = serverName;
    response.headers[ "X-Response-Time" ] = getTickCount() - requestData.startTime;

    // Log response
    logMCPResponse({
        server: serverName,
        statusCode: response.statusCode,
        contentType: response.contentType,
        timestamp: now()
    });

    // Track metrics
    if ( response.statusCode >= 400 ) {
        incrementMetric( "mcp.errors.#serverName#" );
    }
}
```

***

### onMCPError

Fired when an exception occurs during MCP server operations.

**When**: Exception in request handling, class scanning, or other MCP operations **Frequency**: When errors occur

#### Event Arguments

| Argument       | Type        | Description                                               |
| -------------- | ----------- | ------------------------------------------------------------- |
| `server`       | `MCPServer` | The server instance                                       |
| `context`      | `String`    | Where error occurred (`handleRequest`, `scanClass`, etc.) |
| `exception`    | `Struct`    | Exception object (message, detail, stackTrace, type)      |
| `method`       | `String`    | Request method (context: `handleRequest`)                 |
| `requestId`    | `Any`       | Request ID (context: `handleRequest`)                     |
| `params`       | `Struct`    | Request parameters (context: `handleRequest`)             |
| `responseTime` | `Numeric`   | Time elapsed in ms (context: `handleRequest`)              |
| `errorCode`    | `Numeric`   | RPC error code (context: `handleRequest`)                 |
| `classPath`    | `String`    | Class being scanned (context: `scanClass`)                |

#### Example

```javascript
function onMCPError( event, interceptData ) {
    var exception = interceptData.exception;
    var context = interceptData.context;
    var server = interceptData.server;

    // Log detailed error
    writeLog(
        type: "error",
        file: "mcp-errors",
        text: "MCP Error in #context#: #exception.message#\nDetail: #exception.detail#"
    );

    // Context-specific handling
    if ( context == "handleRequest" ) {
        // Request-level error
        var method = interceptData.method;
        var errorCode = interceptData.errorCode;

        // Send alert for critical errors
        if ( errorCode == -32603 ) { // SERVER_ERROR
            emailService.sendAlert(
                to: "ops@example.com",
                subject: "MCP Server Error: #server.getName()#",
                body: "Method: #method#\nError: #exception.message#\nStackTrace: #exception.stackTrace#"
            );
        }

        // Track error metrics
        metrics.increment( "mcp.errors.#method#" );
        metrics.increment( "mcp.errors.code.#errorCode#" );
    } else if ( context == "scanClass" ) {
        // Class scanning error
        writeLog(
            type: "warning",
            file: "mcp-scan-errors",
            text: "Failed to scan class: #interceptData.classPath#"
        );
    }

    // Send to error tracking service
    errorTracker.captureException(
        exception: exception,
        context: {
            mcpServer: server.getName(),
            operation: context,
            serverInfo: server.getServerInfo()
        }
    );
}
```

***

## Client Events

### onMCPClientRequest

Fired before an MCP client sends an HTTP request.

**When**: Before every MCP client HTTP call

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `client` | `MCPClient` | The MCP client instance |
| `baseURL` | string | Server base URL |
| `operation` | string | Operation type (`tool`, `resource`, `prompt`, `discovery`) |
| `name` | string | Tool/resource/prompt name |
| `requestBody` | struct | Request payload |

#### Example

```javascript
BoxRegisterInterceptor( "onMCPClientRequest", function( event ) {
    println( "MCP client #event.operation#: #event.name#" )
})
```

***

### onMCPClientResponse

Fired after an MCP client receives a successful HTTP response.

**When**: On successful MCP client HTTP response

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `client` | `MCPClient` | The MCP client instance |
| `baseURL` | string | Server base URL |
| `operation` | string | Operation type |
| `name` | string | Tool/resource/prompt name |
| `response` | `MCPResponse` | The response object |
| `executionTime` | numeric | Request duration in ms |
| `statusCode` | numeric | HTTP status code |

#### Example

```javascript
BoxRegisterInterceptor( "onMCPClientResponse", function( event ) {
    println( "MCP response: #event.operation#/#event.name# in #event.executionTime#ms" )
})
```

***

### onMCPClientError

Fired when an MCP client encounters an HTTP error or network exception.

**When**: On HTTP errors (bad status / JSON-RPC error) or network exceptions

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `client` | `MCPClient` | The MCP client instance |
| `baseURL` | string | Server base URL |
| `operation` | string | Operation type |
| `name` | string | Tool/resource/prompt name |
| `error` | string | Error message |
| `statusCode` | numeric | HTTP status code (if available) |
| `executionTime` | numeric | Request duration in ms |
| `exception` | any | Exception object (if from catch block) |

#### Example

```javascript
BoxRegisterInterceptor( "onMCPClientError", function( event ) {
    logError( "MCP client error: #event.operation#/#event.name# — #event.error#" )
})
```
