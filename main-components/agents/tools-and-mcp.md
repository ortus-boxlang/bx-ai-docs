---
description: >-
  Tools, the Global Tool Registry, MCP server seeding, and the ClosureTool
  pattern for building AI agents with real-world capabilities.
icon: wrench
---

# Agent Tools & MCP

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

Agents use tools to perform real-world actions — querying databases, calling APIs, running calculations, and more. In v3.0, tools can also be sourced from the Global Tool Registry and from remote MCP servers.

## Basic Tool Usage

```javascript
// Inline lambda tool
weatherTool = aiTool(
    "get_weather",
    "Get current weather for a location",
    location => getWeatherData( location )
).describeLocation( "City and country, e.g. Boston, MA" )

agent = aiAgent(
    name : "WeatherBot",
    tools: [ weatherTool ]
)

response = agent.run( "What's the weather in Paris?" )
// Agent automatically calls get_weather("Paris")
```

## Global Tool Registry (v3.0+)

The Tool Registry is a module-wide singleton that stores tools by name (optionally namespaced by module). Agents can look up tools by name at runtime instead of holding direct references.

```javascript
// Register tools centrally
aiToolRegistry()
    .register( weatherTool, module: "my-app" )
    .register( calculatorTool )

// Resolve tools by name when creating agents
agent = aiAgent(
    name : "Assistant",
    tools: aiToolRegistry().resolveTools( [
        "get_weather@my-app",
        "calculate"
    ] )
)
```

### Registering with Shorthand

```javascript
// Register directly from function reference
aiToolRegistry().register(
    name       : "search_products",
    description: "Search the product catalog",
    callback   : searchProducts
)
```

### `@AITool` Annotation Scanning

Annotate methods with `@AITool` and scan a class or package path:

```javascript
class MyTools {
    @AITool( "Search the product catalog" )
    function search_products( required string query ) {
        return queryProductDB( query )
    }

    @AITool( { name: "get_order", description: "Look up an order by ID" } )
    function get_order( required string orderId ) {
        return queryOrderDB( orderId )
    }
}

// Scan and register all @AITool methods at startup
aiToolRegistry().scan( new MyTools(), module: "my-app" )

// Use in agent
agent = aiAgent(
    name : "SalesBot",
    tools: aiToolRegistry().resolveTools( [ "search_products@my-app", "get_order@my-app" ] )
)
```

### Registry API

| Method | Description |
|---|---|
| `register( tool, module )` | Register an `ITool` instance |
| `register( name, description, callback, module )` | Shorthand — builds `ClosureTool` |
| `scan( instanceOrPath, module )` | Scan `@AITool` annotations |
| `get( key )` | Get a tool by name or `name@module` key |
| `has( key )` | Check if a tool exists |
| `resolveTools( array )` | Convert string keys to `ITool` instances |
| `unregister( key )` | Remove a tool |
| `keys()` | List all registered keys |

See [Tool Registry](../tool-registry.md) for the full reference.

## Built-in Audio Tools (v3.1+) 🎙️

The `bx-ai` module automatically registers three audio tools at startup. Opt in by including their keys in your agent's `tools` array — no registration code required.

| Tool Key | Description |
|---|---|
| `speak@bxai` | Convert text to speech; returns the absolute path to the saved audio file (auto-generates a temp file if no `outputFile` is provided) |
| `transcribe@bxai` | Transcribe a local audio file path or URL to plain text |
| `translate@bxai` | Translate any-language audio to English text |

```javascript
agent = aiAgent(
    name         : "VoiceAssistant",
    instructions : "You are a helpful voice assistant. Speak responses aloud and transcribe audio on request.",
    tools        : [ "now@bxai", "speak@bxai", "transcribe@bxai", "translate@bxai" ]
)

// Agent calls speak@bxai — returns the path to the generated mp3
agent.run( "Welcome the user and tell them today's date" )

// Agent calls transcribe@bxai — returns plain transcript text
agent.run( "Transcribe /recordings/meeting.mp3" )

// Agent calls translate@bxai — returns English translation
agent.run( "Translate the Spanish audio at /audio/mensaje.mp3" )
```

> 💡 The `speak@bxai` tool writes to a temporary file automatically when the agent does not supply an `outputFile`. For production use, instruct the agent to provide a specific output path or post-process the returned path.

See [Audio & Speech](../audio/README.md) for full details on providers, voices, and formats.

## Built-in FileSystem Tools (v3.1+) 📂

`FileSystemTools` gives agents the ability to read, write, move, delete, and list files and directories on the local filesystem. It is **not auto-registered** — you must opt in explicitly, which lets you apply path guards that restrict the AI to specific directories.

### Tool Keys

| Tool Key | Description |
|---|---|
| `readFile@bxai` | Read a file and return its text content |
| `writeFile@bxai` | Write (overwrite) text to a file; creates parent dirs as needed |
| `appendFile@bxai` | Append text to a file; creates the file if it does not exist |
| `fileMetadata@bxai` | Return file metadata (name, size, MIME type, modified date) as JSON |
| `pathExists@bxai` | Check whether a file or directory exists (`"true"` / `"false"`) |
| `deleteFile@bxai` | Delete a file ⚠️ irreversible |
| `moveFile@bxai` | Move or rename a file |
| `copyFile@bxai` | Copy a file to a new location |
| `listDirectory@bxai` | List directory contents as a JSON array; supports glob filters and recursion |
| `createDirectory@bxai` | Create a directory including any missing parent directories |
| `deleteDirectory@bxai` | Recursively delete a directory and all its contents ⚠️ irreversible |

### Registration with Path Guards

```javascript
import bxModules.bxai.models.tools.filesystem.FileSystemTools;

// Restrict to specific directories (strongly recommended)
aiToolRegistry().scanClass(
    new FileSystemTools( allowedPaths: [ "/workspace/data", "/tmp/ai-output" ] ),
    "bxai"
)

// Allow any path — use only in fully-trusted environments
aiToolRegistry().scanClass( new FileSystemTools(), "bxai" )
```

Every path argument is resolved to its canonical form before the guard check runs. This blocks directory-traversal tricks like `../../etc/passwd` even if the AI constructs them.

### Example: Coding Agent

```javascript
agent = aiAgent(
    name         : "CodingAssistant",
    instructions : "You are a coding assistant. You can read, write, and organize project files.",
    tools        : [
        "readFile@bxai",
        "writeFile@bxai",
        "listDirectory@bxai",
        "createDirectory@bxai",
        "fileMetadata@bxai",
        "pathExists@bxai"
    ]
)

// Agent reads a file and rewrites it with improvements
agent.run( "Read src/App.bx and add a descriptive header comment at the top" )

// Agent scans the project for all BoxLang files
agent.run( "List all .bx files under src/ recursively and summarize what each one does" )

// Agent creates an output structure
agent.run( "Create a reports/ directory and write an audit.md file with a summary of findings" )
```

### Example: File listing with glob filter

```javascript
// listDirectory supports glob patterns and recursive search
agent.run(
    "List all JSON and YAML config files under /project/config — use the pattern *.json|*.yaml"
)
```

> 🔐 `deleteFile@bxai` and `deleteDirectory@bxai` are irreversible. Consider omitting them from agents that only need read or write access.

## ClosureTool Pattern

`ClosureTool` wraps a lambda directly without going through `aiTool()`:

```javascript
import bxModules.bxai.models.tools.ClosureTool;

searchTool = new ClosureTool(
    "search_docs",
    "Search documentation",
    query => searchIndex( query )
)

agent = aiAgent(
    name : "DocBot",
    tools: [ searchTool ]
)
```

## Accessing `_chatRequest` in Tools

Tools receive the live `AiChatRequest` as `_chatRequest` in their arguments — useful for reading the current model, adding context, or adjusting behavior:

```javascript
contextTool = aiTool(
    "log_context",
    "Log information about the current request",
    ( args ) => {
        model   = args._chatRequest.getModel()
        history = args._chatRequest.getMessages()
        writeLog( "Tool called during #model# with #history.len()# messages" )
        return "Logged"
    }
)
```

## MCP Server Seeding (v3.0+)

Connect agents to remote MCP servers. Tool names from the server are automatically fetched and made available to the agent:

```javascript
agent = aiAgent(
    name      : "ResearchAgent",
    mcpServers: [
        {
            url      : "http://localhost:3000/mcp",
            toolNames: [ "web_search", "fetch_page" ]
        },
        {
            url      : "http://internal-tools/mcp",
            toolNames: [ "query_crm" ]
        }
    ]
)

// Agent can use web_search, fetch_page, and query_crm from the MCP servers
response = agent.run( "Find recent news about BoxLang" )
```

### MCP Authentication

```javascript
agent = aiAgent(
    name      : "SecureAgent",
    mcpServers: [
        {
            url        : "https://tools-server/mcp",
            toolNames  : [ "search" ],
            bearerToken: server.bearerToken,
            headers    : { "X-Tenant-ID": "acme" }
        }
    ]
)
```

## Dynamic Tool Assignment

```javascript
function getToolsForRole( required string userRole ) {
    if ( userRole == "admin" ) {
        return aiToolRegistry().resolveTools( [ "admin_tool", "user_tool", "report_tool" ] )
    }
    return aiToolRegistry().resolveTools( [ "user_tool" ] )
}

agent = aiAgent( name: "ContextAgent" )
    .setTools( getToolsForRole( currentUser.role ) )
```

## Related Pages

* [Tool Registry](../tool-registry.md) — Full registry reference
* [AI Tools](../tools.md) — `aiTool()` BIF reference
* [Skills](skills.md) — Injecting knowledge vs. calling tools
* [MCP Server](../../advanced/mcp-server.md) — Building your own MCP server
* [MCP Client](../../advanced/mcp-client.md) — Consuming MCP servers directly
