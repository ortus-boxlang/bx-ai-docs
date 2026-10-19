---
description: Create a fluent MCP (Model Context Protocol) client for consuming external MCP servers over HTTP or as a child process over stdio.
icon: server
---

# MCP

Create a fluent MCP (Model Context Protocol) client for consuming external MCP servers. MCP enables AI applications to connect to tools, data sources, and external systems through a standardized protocol.

The argument decides the transport:

* a **URL string** gives an HTTP client (`MCPHttpClient`)
* a **struct with a `command`** gives a stdio client (`MCPStdioClient`) that launches the server as a child process and talks to it over stdin and stdout

Both implement `IMCPClient`, have the same methods, and return the same `MCPResponse`.

## 🔌 MCP Client-Server Flow

```mermaid
sequenceDiagram
    participant App as Your Application
    participant MCP as MCP Client
    participant Server as MCP Server
    participant Res as External Resources

    App->>MCP: MCP( url ) or MCP( { command: [...] } )
    App->>MCP: withTimeout / withBearerToken

    alt Discovery Phase
        App->>MCP: listTools()
        MCP->>Server: tools/list
        Server-->>MCP: Available tools
        MCP-->>App: MCPResponse
    end

    alt Invocation Phase
        App->>MCP: send( name, args )
        MCP->>Server: tools/call
        Server->>Res: Access data/API
        Res-->>Server: Resource data
        Server-->>MCP: Tool result
        MCP-->>App: MCPResponse
    end

    alt Resource Access
        App->>MCP: readResource( uri )
        MCP->>Server: resources/read
        Server-->>MCP: Resource content
        MCP-->>App: MCPResponse
    end

    style MCP fill:#4A90E2
    style Server fill:#7ED321
    style Res fill:#F5A623
```

## Syntax

```javascript
// HTTP
MCP( url )

// Stdio: a child process
MCP( { command: [ "executable", "arg1", "arg2" ], env: {}, cwd: "", inheritEnv: true, maxRestarts: 2, commandGuard: fn, timeout: 30000 } )
```

## Parameters

| Parameter | Type             | Required | Description                                                                                                |
| --------- | ---------------- | -------- | ---------------------------------------------------------------------------------------------------------- |
| `mcpServer` | string \| struct | Yes    | The URL of the MCP server (HTTP), or a struct that describes the program to launch (stdio), see below.     |

### Stdio struct

| Key            | Type     | Default           | Description                                                                                         |
| -------------- | -------- | ----------------- | --------------------------------------------------------------------------------------------------- |
| `command`      | array    | required          | The executable followed by its arguments, one element each. Never a shell string.                   |
| `env`          | struct   | `{}`              | Environment variables added to the child process.                                                   |
| `cwd`          | string   | current directory | Working directory of the child process.                                                             |
| `inheritEnv`   | boolean  | `true`            | `false` starts the child with only `env`.                                                           |
| `maxRestarts`  | numeric  | `2`               | How many times in a row a dead server is started again before the client gives up.                  |
| `commandGuard` | function | none              | `( command ) => boolean`, called before every launch. Return `false` or throw to refuse the command. |
| `timeout`      | numeric  | `30000`           | Milliseconds to wait for each answer.                                                               |

A bad struct throws `InvalidArgument`, and so does an option a stdio server does not have (the HTTP-only ones, or a typo such as `commandGaurd`). A command refused by the guard or by the `mcp.stdio.allowedCommands` setting throws `MCPCommandRefused`.

The first argument used to be called `baseURL`. `MCP( baseURL: url )` still works, the name is now `mcpServer`. See the [MCP Stdio Client](../../../mcp/client/stdio.md) guide.

## Returns

Returns an `IMCPClient`: an `MCPHttpClient` for a URL, an `MCPStdioClient` for a struct. Both have:

* Configuration: `withTimeout()`, `onSuccess()`, `onError()`
* Discovery: `listTools()`, `listResources()`, `listPrompts()`, `getCapabilities()`
* Invocation: `send( tool, args )`, `readResource( uri )`, `getPrompt( name, args )`
* Statistics: `getStats()`, `getSummary()`, `resetStats()`
* Lifecycle: `close()`, a no-op for HTTP that shuts the child process down for stdio

Only `MCPHttpClient` has:

* `withBearerToken()`, `withAuth()`, `withHeaders()`
* Safety controls: `withUrlGuard()`, `withRedirects()`, `withMaxResponseBytes()`, `withHandshake()`, `withHttpVersion()`, see [Controlling What a Client Can Reach](../../../mcp/client/README.md#controlling-what-a-client-can-reach)

Only `MCPStdioClient` has `isRunning()`, `getProtocolVersion()` and `getStderrTail()`.

### The MCPResponse

Every call returns an `MCPResponse`. Nothing throws from a call, a failure is a response with `isSuccess()` false.

| Method            | Description                                                              |
| ----------------- | ------------------------------------------------------------------------ |
| `isSuccess()`     | `true` when the server answered with a result                            |
| `hasError()`      | `true` when the call failed                                              |
| `getData()`       | The result of the call, an empty struct on failure                       |
| `getError()`      | The error message, empty on success                                      |
| `getStatusCode()` | The HTTP status. For stdio, `200` for an answer and `0` when none came   |
| `getHeaders()`    | The HTTP response headers, empty for stdio                               |
| `onSuccess( fn )` / `onError( fn )` | Run a function when the response is a success or an error |

## Examples

### Basic MCP Client

```javascript
// Connect to MCP server
client = MCP( "http://localhost:3000" );

// Call a tool
result = client.send( "searchDocs", { query: "syntax" } );
println( result.getData() );
```

### A Stdio Server

```javascript
// Launch a server as a child process
client = MCP( { command: [ "bxplaywright", "mcp" ], env: { HEADLESS: "true" } } );

result = client.send( "browser_visit", { url: "https://boxlang.io" } );
if ( result.isSuccess() ) {
    println( result.getData() );
} else {
    println( "Failed: #result.getError()#" );
}

// Stop the child process
client.close();
```

### With Configuration

```javascript
// Configure an HTTP client with options
client = MCP( "http://localhost:3000" )
    .withTimeout( 5000 )
    .withBearerToken( "my-secret-token" )
    .withHeaders({
        "X-Client-ID": "boxlang-app",
        "X-Request-ID": createUUID()
    });

result = client.send( "getTasks", {} );
```

### Callback Handlers

```javascript
// Add success and error handlers
client = MCP( "http://localhost:3000" )
    .onSuccess( ( response ) => {
        writeLog( text: "Success: #response.getData()#", type: "information", log: "ai" );
    })
    .onError( ( response ) => {
        writeLog( text: "Error: #response.getError()#", type: "error", log: "ai" );
    });

client.send( "processData", { input: "test" } );
```

### Discover Tools

`listTools()` returns an `MCPResponse`, the tools are in `getData().tools`.

```javascript
client = MCP( "http://localhost:3000" );

response = client.listTools();
if ( response.isSuccess() ) {
    println( "Available Tools:" );
    response.getData().tools.each( tool => {
        println( "  - #tool.name#: #tool.description#" );
    });
}
```

A server that pages its tools answers with a `nextCursor`. Pass it to `listTools( cursor )` to read the next page.

### Discover Resources

```javascript
client = MCP( "http://localhost:3000" );

response = client.listResources();
response.getData().resources.each( resource => {
    println( "  - #resource.uri#: #resource.name#" );
});
```

### Read a Resource

```javascript
client = MCP( "http://localhost:3000" );

resource = client.readResource( "docs://readme" );
println( resource.getData() );
```

### Discover Prompts

```javascript
client = MCP( "http://localhost:3000" );

response = client.listPrompts();
response.getData().prompts.each( prompt => {
    println( "  - #prompt.name#: #prompt.description#" );
});
```

### Get a Prompt

```javascript
client = MCP( "http://localhost:3000" );

prompt = client.getPrompt( "greeting", {
    name: "John",
    language: "Spanish"
});

messages = prompt.getData().messages;
```

### Error Handling

```javascript
client = MCP( "http://localhost:3000" );

result = client.send( "riskyOperation", { input: data } );

if ( result.hasError() ) {
    println( "MCP Error: #result.getError()#" );
} else {
    println( "Success: #result.getData()#" );
}
```

### Timeout Configuration

```javascript
// Set custom timeout for long operations
client = MCP( "http://localhost:3000" )
    .withTimeout( 30000 ); // 30 seconds, in milliseconds

result = client.send( "longRunningTask", { input: largeDataset } );
```

### Integration with AI Agents

The easiest way to use the tools of an MCP server is to give the server to an agent or a model. It lists the tools, wraps each one, and calls them for you.

```javascript
agent = aiAgent(
    name: "ops",
    model: aiModel( provider: "openai" ),
    mcpServers: [
        "http://localhost:3000",
        { command: [ "bxplaywright", "mcp" ], prefix: "browser", toolFilter: [ "browser_visit" ] }
    ]
);

response = agent.run( "Search for BoxLang docs" );

// Release the child processes
agent.closeMCPServers();
```

See [Tools & MCP](../../../main-components/agents/tools-and-mcp.md) for `prefix`, `toolFilter` and the rest of the options.

You can also build the tools yourself from a client:

```javascript
client = MCP( "http://localhost:3000" );

mcpTools = client.listTools().getData().tools;

aiTools = mcpTools.map( mcpTool => {
    return aiTool(
        mcpTool.name,
        mcpTool.description,
        ( args ) => client.send( mcpTool.name, args ).getData()
    );
});

agent = aiAgent( name: "mcp-agent", model: aiModel( provider: "openai" ), tools: aiTools );
```

### Multiple MCP Servers

```javascript
// Connect to multiple servers
docsServer = MCP( "http://docs.example.com:3000" );
apiServer = MCP( "http://api.example.com:3001" );

// Use different servers
docs = docsServer.send( "search", { query: "API" } );
data = apiServer.send( "getData", { id: 123 } );
```

### Health Check

A failed call does not throw, check the response.

```javascript
function checkMCPHealth( baseURL ) {
    return MCP( baseURL ).withTimeout( 2000 ).listTools().isSuccess();
}

if ( checkMCPHealth( "http://localhost:3000" ) ) {
    println( "MCP server is available" );
} else {
    println( "MCP server is unavailable" );
}
```

### Retry Logic

```javascript
// Implement retry logic for MCP calls
function mcpCallWithRetry( client, toolName, args, maxRetries = 3 ) {
    for ( var attempt = 1; attempt <= maxRetries; attempt++ ) {
        var result = client.send( toolName, args );

        if ( result.isSuccess() ) {
            return result.getData();
        }

        if ( attempt < maxRetries ) {
            sleep( 1000 * attempt ); // Linear backoff
        }
    }

    throw( type: "MCPCallFailed", message: "MCP call failed after #maxRetries# attempts" );
}
```

{% hint style="warning" %}
Do not retry a tool call that changes something, such as a purchase or a file write, unless the tool is safe to repeat. The stdio client never repeats a call by itself for this reason.
{% endhint %}

### Dynamic Tool Discovery

```javascript
client = MCP( "http://localhost:3000" );

// Find a tool by name pattern
tools = client.listTools().getData().tools;
searchTools = tools.filter( t => t.name.findNoCase( "search" ) );

if ( searchTools.len() ) {
    result = client.send( searchTools[ 1 ].name, { query: "BoxLang" } );
    println( result.getData() );
}
```

### Caching MCP Responses

```javascript
// Cache MCP tool responses
function callMCPWithCache( client, toolName, args, ttl = 60 ) {
    cacheKey = "mcp_#toolName#_#hash( jsonSerialize( args ) )#";

    if ( cacheExists( cacheKey ) ) {
        return cacheGet( cacheKey );
    }

    data = client.send( toolName, args ).getData();

    cachePut( cacheKey, data, ttl );
    return data;
}

client = MCP( "http://localhost:3000" );
data = callMCPWithCache( client, "expensiveTool", { id: 123 }, 300 );
```

## Notes

* 🔌 **Protocol**: Implements the Model Context Protocol standard for AI integrations
* 🌐 **Two transports**: HTTP/HTTPS for remote servers, stdio for programs you launch
* 🔧 **Fluent API**: Chainable configuration methods
* 🎯 **Discovery**: Dynamically discover tools, resources, and prompts
* 🔐 **Security**: Bearer tokens, custom headers and URL controls for HTTP. An allow list and a command guard for stdio
* ⚡ **Configurable**: Timeout and header customization
* 📡 **Callbacks**: Success and error handlers

## Related Functions

* [`MCPServer()`](mcpserver.md) - Create MCP server to expose tools
* [`aiTool()`](aitool.md) - Create AI tools (can wrap MCP tools)
* [`aiAgent()`](aiagent.md) - Use MCP tools with agents
* [MCP Stdio Client](../../../mcp/client/stdio.md) - Launch a server as a child process

## Best Practices

✅ **Discover before using** - List tools/resources to understand capabilities

✅ **Check the response** - Calls do not throw, look at `isSuccess()`

✅ **Use timeouts** - Set appropriate timeouts for network operations

✅ **Secure connections** - Use HTTPS and bearer tokens for production

✅ **Close stdio clients** - Call `close()`, or `agent.closeMCPServers()`

✅ **Cache responses** - Cache expensive MCP calls when appropriate

❌ **Don't hardcode URLs** - Use environment variables for MCP server URLs

❌ **Don't ignore errors** - Always check the response before using data

❌ **Don't build a command from user input** - A stdio client runs a program

❌ **Don't skip authentication** - Use tokens for protected MCP servers
