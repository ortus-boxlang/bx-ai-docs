---
description: >-
  A centralized registry for discovering, registering, and resolving AI tools
  across modules and classes.
icon: toolbox
---

# Tool Registry

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

The **Tool Registry** is a singleton that manages AI tools across your entire application. Instead of manually passing tool arrays to every agent, you can register tools once and resolve them by name anywhere.

## Why Use the Registry?

| Without Registry | With Registry |
| --- | --- |
| Pass tools array to every agent/model call | Register once, reference by name |
| No cross-module tool sharing | Tools shared across modules |
| Manual `@AITool` discovery | Auto-scan classes for annotated methods |
| Duplicate tool definitions | Single source of truth |

## Accessing the Registry

```javascript
// Get the singleton registry
registry = aiToolRegistry()
```

## Registering Tools

### Using our `aiTool()` BIF

All tools that you create with `aiTool()` are automatically registered in the global registry.  If you don't want them to be automatically registered, you can set `register: false` in the options and register manually with `aiToolRegistry().register()`.

```javascript
// Positional args
myTool = aiTool( "search", "Search the database", ( required query ) => db.search( query ) )

// Named args
myTool = aiTool(
    name       : "search",
    description: "Search the database",
    callback   : ( required query ) => db.search( query )
)
```

### Tool Registry Registration

You can also register any closure or `ITool` instance directly with the registry, no `aiTool()` wrapper needed:

```javascript
// Register a closure directly — no aiTool() wrapper needed
aiToolRegistry().register(
    name       : "calculate",
    description: "Perform math calculations",
    callback   : ( expression ) => mathCall( expression )
)
```

### With Module Attribution

Use a module name to group tools for easy bulk unregistration later:

```javascript
aiToolRegistry().register( myTool, "my-module" )
// Key becomes: "search@my-module"
```

### Scanning a Class with `@AITool`

Annotate methods with `@AITool` and scan the class to register all at once:

```javascript
class CustomerService {

    @AITool( description = "Look up customer by ID or email" )
    function lookupCustomer( required string identifier ) {
        return queryExecute( "SELECT * FROM customers WHERE id = :id OR email = :id", { id: identifier } )
    }

    @AITool( description = "Create a support ticket" )
    function createTicket( required string issue, string priority = "normal" ) {
        return ticketService.create( issue, priority )
    }

    function internalHelper() {  // Not annotated — not registered
        ...
    }
}

// Register all @AITool methods from the class
aiToolRegistry().scan( new CustomerService(), "customer-module" )
```

### Scanning a Package Path

```javascript
// Scan all .bx files in a package for @AITool annotations
aiToolRegistry().scan( "com.myapp.tools", "my-module" )
```

### @AITool Options

The `@AITool` annotation supports the following formats for defining the tool name and description:

* Single `@AITool` : The `name` is derived from the method name, and the `description` is taken from the method hint
* `@AITool( "Description string" )` : The method name is used as the tool name, and the provided string is used as the description
* `@AITool( { name: "x", description: "y" } )` : Explicitly specify both the tool name and description

Example:

```js
@AITool
@AITool( "Description string" )
@AITool( { name: "x", description: "y" } )
```

## Using Registered Tools

### Resolve by Name

```javascript
// Get a single tool by key
searchTool = aiToolRegistry().get( "search" )
// or with module:
searchTool = aiToolRegistry().get( "search@my-module" )

// Check existence before use
if ( aiToolRegistry().has( "calculate" ) ) {
    calculatorTool = aiToolRegistry().get( "calculate" )
}
```

### Resolve an Array (Mix of Strings and Instances)

`resolveTools()` lets you pass a heterogeneous array of tool names and/or `ITool` instances:

```javascript
agent = aiAgent(
    name : "Assistant",
    tools: aiToolRegistry().resolveTools( [
        "search",              // Registry key → resolved to ITool
        "calculate@my-module", // Keyed key
        customInlineTool       // ITool instance → passed through
    ] )
)
```

### Introspect the Registry

```javascript
registry = aiToolRegistry()

// List all registered keys
println( registry.keys() )

// Detailed info about a specific tool
info = registry.getToolInfo( "search" )
println( info.name )
println( info.description )

// Full listing
all = registry.listTools()
```

## Unregistering Tools

```javascript
// Remove a single tool
aiToolRegistry().unregister( "search" )

// Remove all tools from a module (bulk cleanup)
aiToolRegistry().unregisterByModule( "my-module" )
```

## Built-in Tools (v3.0+)

The module ships with several built-in tool classes that are auto-registered in the global tool registry at startup. You can opt-in to these tools by name when creating agents.

### Core Tools

| Tool Key | Description |
|---|---|
| `httpGet@bxai` | Fetch the contents of a URL via HTTP GET |
| `log@bxai` | Write a message to the `ai.log` file (`info`, `warning`, `error`, etc.) |
| `now@bxai` | Return the current date/time in ISO 8601 format |
| `print@bxai` | Print a message to the console (debug/output visibility) |
| `sendEmail@bxai` | Send an email using the server's configured mail service |

```javascript
// Use auto-registered core tools directly
agent = aiAgent( tools: [ "print@bxai", "log@bxai", "now@bxai", "httpGet@bxai" ] )
```

### Audio Tools

| Tool Key | Description |
|---|---|
| `speak@bxai` | Convert text to speech, save to file, return the file path |
| `transcribe@bxai` | Transcribe a local file or URL to plain text |
| `translate@bxai` | Translate any-language audio to English text |

```javascript
agent = aiAgent( tools: [ "speak@bxai", "transcribe@bxai", "translate@bxai" ] )
```

### Image Tools (v3.2.0+)

| Tool Key | Description |
|---|---|
| `generateImage@bxai` | Generate an image from a text prompt, save to file, return the file path |

```javascript
agent = aiAgent( tools: [ "generateImage@bxai" ] )
```

### Filesystem Tools

| Tool Key | Description |
|---|---|
| `readFile@bxai` | Read file contents |
| `readMultipleFiles@bxai` | Read multiple files at once |
| `writeFile@bxai` | Write content to a file |
| `appendFile@bxai` | Append content to a file |
| `editFile@bxai` | Edit a file with search/replace |
| `fileMetadata@bxai` | Get file metadata |
| `pathExists@bxai` | Check if a path exists |
| `deleteFile@bxai` | Delete a file |
| `moveFile@bxai` | Move a file |
| `copyFile@bxai` | Copy a file |
| `searchFiles@bxai` | Search for files by pattern |
| `listAllowedDirectories@bxai` | List allowed directories |
| `listDirectory@bxai` | List directory contents |
| `directoryTree@bxai` | Get directory tree structure |
| `createDirectory@bxai` | Create a directory |
| `deleteDirectory@bxai` | Delete a directory |
| `zipFiles@bxai` | Create a zip archive |
| `unzipFile@bxai` | Extract a zip archive |
| `checkZipFile@bxai` | Validate a zip file |

> 🚨 **Filesystem tools are NOT auto-registered.** They must be explicitly enabled via `aiToolRegistry().scanClass()` so agents never get filesystem access unless explicitly granted. Supports a path-guard constructor (`allowedPaths: [...]`) that blocks directory-traversal attacks.

```javascript
// Opt-in to filesystem tools
aiToolRegistry().scanClass( "bxModules.bxai.models.tools.filesystem.FileSystemTools", "my-app" )

// Use in agent
agent = aiAgent( tools: aiToolRegistry().resolveTools( [ "readFile@bxai", "writeFile@bxai" ] ) )
```

## Web Search Tools (v3.2.0+)

| Tool Key | Description |
|---|---|
| `webSearch@bxai` | Searches the web for information and returns an array of results (title, URL, snippet, ...) |

```javascript
// Auto-registered at module startup
agent = aiAgent( tools: [ "webSearch@bxai" ] )
```

> 💡 `provider = ""` means "use the configured default provider" (HTTP by default). `maxResults = 0` means "use provider/module defaults".



## Agent Registry (v3.2.0+)

In addition to the Tool Registry, BoxLang AI 3.2.0 introduces the **Agent Registry** — a singleton for centralized agent management.

```javascript
// Register an agent
agent = aiAgent(
    name: "support-agent",
    description: "Customer support agent",
    register: true,
    module: "my-app"
)

// Or register manually
aiAgentRegistry().register( agent, "my-app" )

// List all registered agents
agents = aiAgentRegistry().listAgents()

// Resolve agents from mixed array
resolved = aiAgentRegistry().resolveAgents( [
    "support-agent@my-app",
    anotherAgentInstance
] )

// Get agent info
info = aiAgentRegistry().getAgentInfo( "support-agent@my-app" )

// Unregister
aiAgentRegistry().unregister( "support-agent@my-app" )
aiAgentRegistry().unregisterByModule( "my-app" )
```

### Agent Registry API

| Method | Description |
|---|---|
| `register( agent, module )` | Register an agent with optional module namespace |
| `unregister( key )` | Remove agent by key |
| `unregisterByModule( module )` | Remove all agents from a module |
| `resolveAgents( array )` | Resolve mixed array of string keys and `AiAgent` instances |
| `listAgents()` | Return struct of all registered agents |
| `getAgentInfo( key )` | Return `{ name, description, module }` for a key |

### Agent Registry Events

| Event | When Fired |
|---|---|
| `onAIAgentRegistryRegister` | Agent registered |
| `onAIAgentRegistryUnregister` | Agent unregistered |

## Related Pages

* [Tools & MCP (Agents)](agents/tools-and-mcp.md) — Using the registry with agents
* [aiToolRegistry() Reference](../advanced/reference/built-in-functions/aitoolregistry.md) — BIF reference
* [Tools](tools.md) — Core tool concepts and `aiTool()` usage
* [Custom Tools](../extending-boxlang-ai/custom-tools.md) — Building custom tool classes
