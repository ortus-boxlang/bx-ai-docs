---
icon: wand-magic-sparkles
---

# Annotation-Based Discovery

Automatically discover and register tools, resources, and prompts from annotated methods.

## Overview

Instead of manually calling `.registerTool()` for each method, use annotations to mark methods as MCP capabilities. Then use `scan()` or `scanClass()` to auto-register them.

## Two Discovery Methods

| Method | Purpose | Use Case |
|---|---|---|
| `scan( path, recurse )` | Scan a package/directory for annotated `.bx` files | Large apps with organized folder structure |
| `scanClass( classPath )` | Scan a single class | Quick discovery or specific classes |

## scan() — Packages & Directories

Automatically discover all annotated methods in a package or folder (supports recursion).

```javascript
// Dot-notation package path
MCPServer( "myApp" ).scan( "models.tools" )

// Dot-notation with recursion disabled
MCPServer( "myApp" ).scan( "models.tools", false )

// Absolute directory path
MCPServer( "myApp" ).scan( "/path/to/tools/" )

// Relative directory path
MCPServer( "myApp" ).scan( "./tools" )
```

## scanClass() — Single Classes

Target one specific class (existing instance, dot-path, or file path).

```javascript
// Already-instantiated object
MCPServer( "myApp" ).scanClass( new models.tools.MyTools() )

// Dot-notation class path (no-arg constructor required)
MCPServer( "myApp" ).scanClass( "models.tools.MyTools" )

// Absolute file path
MCPServer( "myApp" ).scanClass( "/path/to/MyTools.bx" )

// Relative file path
MCPServer( "myApp" ).scanClass( "./tools/MyTools.bx" )
```

## Mixing scan() and scanClass()

Both methods are chainable:

```javascript
MCPServer( "myApp" )
    .scan( "models.tools" )                        // whole package
    .scanClass( new models.special.CustomTool() )  // one extra class
    .scan( "./legacy/tools", false )               // non-recursive folder
```

## @mcpTool Annotation

Register methods as MCP tools:

```javascript
class {

    /**
     * Search for documents
     * @query The search query
     */
    @mcpTool
    function search( required string query ) {
        return searchService.find( query )
    }

    /**
     * @query The query to search
     */
    @mcpTool( "Search for documents in the knowledge base" )
    function searchDocs( required string query ) {
        return docService.search( query )
    }

    @mcpTool( { name: "calculator", description: "Perform calculations", version: "2.0.0" } )
    function calculate( required string expression ) {
        return evaluate( expression )
    }

}
```

**Annotation Formats:**

- `@mcpTool` — Name from method name, description from function hint
- `@mcpTool( "Description" )` — Custom description
- `@mcpTool( { name: "...", description: "...", version: "..." } )` — All custom

## @mcpResource Annotation

Register methods as MCP resources:

```javascript
class {

    /**
     * Returns the README file
     */
    @mcpResource
    function readme() {
        return fileRead( "/readme.md" )
    }

    @mcpResource( "API documentation for developers" )
    function apiDocs() {
        return generateApiDocs()
    }

    @mcpResource( { uri: "config://app", name: "App Config", description: "Application settings", mimeType: "application/json" } )
    function getConfig() {
        return application.settings
    }

}
```

**Annotation Formats:**

- `@mcpResource` — URI and name from method name
- `@mcpResource( "Description" )` — Custom description
- `@mcpResource( { uri: "...", name: "...", description: "...", mimeType: "..." } )` — All custom

## @mcpPrompt Annotation

Register methods as MCP prompts:

```javascript
class {

    /**
     * Generate a greeting message
     * @name The person's name
     */
    @mcpPrompt
    function greeting( required string name ) {
        return [
            { role: "system", content: "You are a friendly assistant." },
            { role: "user", content: "Say hello to #name#" }
        ]
    }

    @mcpPrompt( "Generate code based on a description" )
    function codeGen( required string description, string language = "java" ) {
        return [
            { role: "system", content: "You are a code generator for #language#." },
            { role: "user", content: description }
        ]
    }

    @mcpPrompt( { name: "reviewer", description: "Code review prompt", arguments: [ { name: "code", required: true } ] } )
    function reviewCode( required string code ) {
        return [
            { role: "system", content: "Review this code for issues." },
            { role: "user", content: code }
        ]
    }

}
```

**Annotation Formats:**

- `@mcpPrompt` — Name from method name
- `@mcpPrompt( "Description" )` — Custom description
- `@mcpPrompt( { name: "...", description: "...", arguments: [...] } )` — All custom

## Complete Example

### Project Structure

```
models/tools/
├── SearchTools.bx
├── AdminTools.bx
└── GeneratorTools.bx
```

### SearchTools.bx

```javascript
class {

    @mcpTool( "Search products by name or category" )
    function searchProducts( required string query, string category = "" ) {
        return productService.search(
            query: arguments.query,
            category: arguments.category
        )
    }

    @mcpTool( "Get detailed product information" )
    function getProduct( required string productId ) {
        return productService.getById( arguments.productId )
    }

}
```

### AdminTools.bx

```javascript
class {

    @mcpTool( "Create a new user account" )
    function createUser( required string email, required string name ) {
        return userService.create( email: arguments.email, name: arguments.name )
    }

    @mcpTool( "Delete a user account" )
    function deleteUser( required string userId ) {
        return userService.delete( arguments.userId )
    }

}
```

### Application.bx

```javascript
class {

    function onApplicationStart() {
        // Scan entire package and auto-register all tools
        MCPServer( "myApp" )
            .setDescription( "My Application with Auto-Discovered Tools" )
            .setVersion( "1.0.0" )
            .scan( "models.tools" )  // Auto-discovers all @mcpTool methods
    }

}
```

After `scan()`, all methods annotated with `@mcpTool`, `@mcpResource`, or `@mcpPrompt` are automatically registered!

## Benefits of Annotation-Based Discovery

✅ **Cleaner Code** — Keep tool definitions next to implementations
✅ **Less Boilerplate** — No repetitive `.registerTool()` calls
✅ **Scalability** — Add new tools without updating registration code
✅ **Organization** — Group related tools in classes
✅ **Documentation** — Hints and documentation preserved in code

## Next Steps

- 🏗️ [Class-Based Servers](class-based-servers.md) — Extend MCPServer class
- 🧩 [Tool Registration](registration.md) — Manual registration (inline approach)
- 💡 [Examples](/_examples.md) — Complete working projects
