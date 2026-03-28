---
description: >-
  Build reusable, annotated, and class-based tools that your AI agents can
  call during conversations.
icon: wrench-simple
---

# Custom Tools

Tools give agents the ability to call your application code during a conversation. This page covers all the ways to create custom tools — from quick inline closures to full class hierarchies.

## Quick Tool via aiTool()

The fastest way to create a tool is with the `aiTool()` BIF:

```javascript
searchTool = aiTool(
    name        : "searchProducts",
    description : "Search the product catalog",
    parameters  : [
        { name: "query",      type: "string",  description: "Search term",       required: true },
        { name: "maxResults", type: "number",  description: "Max results to return", required: false }
    ],
    callback    : ( query, maxResults = 10 ) => productService.search( query, maxResults )
)

agent = aiAgent( name: "shop-assistant", tools: [ searchTool ] )
```

> See [aiTool() Reference](../advanced/reference/built-in-functions/aitool.md) for full parameter docs.

## Accessing Conversation Context (`_chatRequest`)

Closures receive `_chatRequest` injected into their arguments so you can access the full conversation context:

```javascript
contextAwareTool = aiTool(
    name        : "getPersonalizedRecommendations",
    description : "Get recommendations based on the conversation so far",
    parameters  : [
        { name: "category", type: "string", description: "Product category", required: true }
    ],
    callback    : function( category, _chatRequest ) {
        // _chatRequest is the full AiChatRequest — read conversation history
        var messages = _chatRequest.getMessages()
        var history  = messages.map( m => m.content ).toList( " " )
        return recommendationEngine.suggest( category, history )
    }
)
```

## Register via Tool Registry

Instead of passing tools to each agent individually, register them globally:

```javascript
aiToolRegistry().register(
    name       : "translateText",
    description: "Translate text to another language",
    callback   : ( text, targetLanguage ) => translationService.translate( text, targetLanguage )
)

// Later — resolve by name
agent = aiAgent(
    name : "multilingual-agent",
    tools: aiToolRegistry().resolveTools( [ "translateText", "searchProducts" ] )
)
```

> See [Tool Registry](../main-components/tool-registry.md) for the full registry API.

## Annotated Class Tools (`@AITool`)

Annotate methods with `@AITool` and scan the class — no manual registration needed:

```javascript
class OrderService {

    @AITool( "Look up order status by order ID" )
    function getOrderStatus( required string orderId ) {
        return queryExecute( "SELECT status FROM orders WHERE id = :id", { id: orderId } )
    }

    @AITool( { name: "cancelOrder", description: "Cancel an order that hasn't shipped" } )
    function cancelOrder( required string orderId, string reason = "" ) {
        return orderRepo.cancel( orderId, reason )
    }

    // Not annotated — not visible to the registry
    private any function internalHelper() { ... }
}

// Register all @AITool methods from the class
aiToolRegistry().scan( new OrderService(), "orders-module" )
```

**Annotation formats:**

| Format | Effect |
| --- | --- |
| `@AITool` | Uses method name + `@hint` as description |
| `@AITool( "Description" )` | Explicit description string |
| `@AITool( { name: "x", description: "y" } )` | Full struct with custom name |

## Extending BaseTool

For maximum control — parameter validation, custom schema, fluent API — extend `BaseTool`:

```javascript
import bxModules.bxai.models.tools.BaseTool;

class extends="BaseTool" {

    function init() {
        variables.name        = "executeSQL"
        variables.description = "Run a SELECT query and return results as JSON"
        return this
    }

    /**
     * Required: implement the actual tool logic.
     * @args The tool arguments passed by the AI
     * @chatRequest The originating chat request (optional access)
     */
    any function doInvoke( required struct args, any chatRequest ) {
        if ( !args.keyExists( "query" ) ) {
            throw( type: "InvalidInput", message: "query argument is required" )
        }
        // Only allow SELECT statements
        if ( !args.query.trim().upper().startsWith( "SELECT" ) ) {
            throw( type: "InvalidInput", message: "Only SELECT queries are allowed" )
        }
        return queryExecute( args.query ).toJSON()
    }

    /**
     * Required: generate the JSON schema for this tool.
     * Return an OpenAI-compatible function schema.
     */
    struct function generateSchema() {
        return {
            "type": "function",
            "function": {
                "name"        : variables.name,
                "description" : variables.description,
                "parameters"  : {
                    "type"      : "object",
                    "properties": {
                        "query": {
                            "type"       : "string",
                            "description": "SQL SELECT statement to execute"
                        }
                    },
                    "required": [ "query" ]
                }
            }
        }
    }
}

// Use it
sqlTool = new tools.ExecuteSQLTool()
agent   = aiAgent( name: "data-analyst", tools: [ sqlTool ] )
```

### Fluent Schema Helpers

`BaseTool` provides helper methods for building schemas without writing JSON manually:

```javascript
class extends="BaseTool" {

    function init() {
        variables.name        = "sendNotification"
        variables.description = "Send a notification to a user"

        // Describe args — used in auto-generated schema
        describeArg( "userId",  "The unique identifier of the recipient" )
        describeArg( "message", "Notification message text" )
        describeArg( "channel", "Delivery channel: email, sms, or push" )

        return this
    }

    any function doInvoke( required struct args, any chatRequest ) {
        return notificationService.send( args.userId, args.message, args.channel ?: "email" )
    }

    struct function generateSchema() {
        // Use BaseTool's auto-schema builder (reads argDescriptions)
        return super.generateSchema()
    }
}
```

## Tool Lifecycle Events

Every tool invocation fires BoxLang interceptor events:

```javascript
// Intercept all tool executions (e.g., for auditing)
BoxRegisterInterceptor( "beforeAIToolExecute", function( event ) {
    auditLog.write( "Tool called: #event.toolName# with args: #event.args.toJSON()#" )
})

BoxRegisterInterceptor( "afterAIToolExecute", function( event ) {
    metricsService.record( "tool.#event.toolName#.duration", event.duration )
})
```

| Event | Fires When | Context Keys |
| --- | --- | --- |
| `beforeAIToolExecute` | Before tool executes | `tool`, `toolName`, `args`, `chatRequest` |
| `afterAIToolExecute` | After tool executes | `tool`, `toolName`, `args`, `result`, `duration` |

## Related Pages

* [Tools](../main-components/tools.md) — Using tools with agents and models
* [Tool Registry](../main-components/tool-registry.md) — Register and resolve tools globally
* [Agents — Tools & MCP](../main-components/agents/tools-and-mcp.md) — Agent-specific tool patterns
* [aiTool() Reference](../advanced/reference/built-in-functions/aitool.md) — BIF reference
