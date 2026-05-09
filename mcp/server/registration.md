---
icon: list-check
---

# Registering Tools, Resources & Prompts

Register capabilities that your MCP server exposes to clients.

## Tool Registration

### Register a Single Tool

```javascript
server = MCPServer( "myApp" )
    .registerTool(
        aiTool( "getWeather", "Get current weather for a location", ( location ) => {
            return weatherService.getCurrent( location )
        } )
        .describeArg( "location", "City name or coordinates" )
    )
```

### Register Multiple Tools

```javascript
tools = [
    aiTool( "search", "Search documents", searchHandler ),
    aiTool( "translate", "Translate text", translateHandler ),
    aiTool( "summarize", "Summarize text", summarizeHandler )
]

server = MCPServer( "myApp" )
    .registerTools( tools )
```

### Check & Retrieve Tools

```javascript
// Check if a tool exists
exists = server.hasTool( "search" )

// Get a specific tool
tool = server.getTool( "search" )

// Get tool count
count = server.getToolCount()

// Get all tools
tools = server.getTools()

// List tools (MCP format)
toolsList = server.listTools()
// [
//     {
//         name: "search",
//         description: "Search documents",
//         inputSchema: { ... }
//     }
// ]
```

### Unregister Tools

```javascript
// Remove a specific tool
server.unregisterTool( "oldTool" )

// Remove all tools
server.clearTools()
```

## Resource Registration

Resources provide access to documents and data.

### Register a Resource

```javascript
server = MCPServer( "myApp" )
    .registerResource(
        uri: "docs://readme",
        name: "README",
        description: "Project documentation",
        mimeType: "text/markdown",
        handler: () => {
            return fileRead( expandPath( "/readme.md" ) )
        }
    )
```

### Register Dynamic Resources

```javascript
// Database content
server.registerResource(
    uri: "db://users",
    name: "User List",
    description: "Current user data",
    mimeType: "application/json",
    handler: () => {
        return userService.getAllUsers()
    }
)

// Configuration
server.registerResource(
    uri: "config://app",
    name: "App Configuration",
    description: "Application settings",
    mimeType: "application/json",
    handler: () => {
        return application.settings
    }
)
```

### List and Read Resources

```javascript
// List available resources
resources = server.listResources()

// Check if resource exists
exists = server.hasResource( "docs://readme" )

// Read a resource
content = server.readResource( "docs://readme" )
// {
//     contents: [
//         {
//             uri: "docs://readme",
//             mimeType: "text/markdown",
//             text: "# Readme content..."
//         }
//     ]
// }
```

### Manage Resources

```javascript
// Remove a resource
server.unregisterResource( "docs://readme" )

// Clear all resources
server.clearResources()
```

## Prompt Registration

Prompts provide reusable prompt templates for AI clients.

### Register a Prompt

```javascript
server = MCPServer( "myApp" )
    .registerPrompt(
        name: "codeReview",
        description: "Code review prompt template",
        args: [
            { name: "language", description: "Programming language", required: true },
            { name: "code", description: "Code to review", required: true }
        ],
        handler: ( args ) => {
            return [
                {
                    role: "system",
                    content: "You are a code reviewer specializing in #args.language#."
                },
                {
                    role: "user",
                    content: "Please review this code:\n\n```#args.language#\n#args.code#\n```"
                }
            ]
        }
    )
```

### List and Get Prompts

```javascript
// List available prompts
prompts = server.listPrompts()

// Check if prompt exists
exists = server.hasPrompt( "codeReview" )

// Get a prompt with arguments
result = server.getPrompt( "codeReview", {
    language: "java",
    code: "public void test() {}"
} )
// {
//     description: "Code review prompt template",
//     messages: [
//         { role: "system", content: { type: "text", text: "..." } },
//         { role: "user", content: { type: "text", text: "..." } }
//     ]
// }
```

## Fluent Registration Pattern

All registration methods return `this` for chaining:

```javascript
server = MCPServer( "myApp" )
    .setDescription( "Complete server" )
    .setVersion( "1.0.0" )
    
    // Register tools
    .registerTool( searchTool )
    .registerTool( calculateTool )
    
    // Register resources
    .registerResource(
        uri: "docs://readme",
        name: "README",
        description: "Documentation",
        mimeType: "text/markdown",
        handler: () => fileRead( "./readme.md" )
    )
    
    // Register prompts
    .registerPrompt(
        name: "reviewer",
        description: "Code review",
        args: [ { name: "code", required: true } ],
        handler: ( args ) => [
            { role: "system", content: "Review this code." },
            { role: "user", content: args.code }
        ]
    )
    
    // Configure security
    .withBasicAuth( "admin", "secret" )
    .withCors( "https://example.com" )
```

## Next Steps

- 📋 [Annotation-Based Discovery](annotation-discovery.md) — Auto-register with @mcpTool
- 🏗️ [Class-Based Servers](class-based-servers.md) — Organize in a class
- 💡 [Examples](/_examples.md) — Complete working examples
