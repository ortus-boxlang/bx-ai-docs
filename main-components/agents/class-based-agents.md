---
description: >-
  Build reusable, encapsulated agents by extending the AiAgent class instead of
  configuring everything inline with the aiAgent() BIF.
icon: boxes-stacked
---

# Class-Based Agents

For simple workflows, `aiAgent()` is perfect. For larger applications, teams often prefer **encapsulated agent classes** that bundle tools, memory, model settings, and conventions in one reusable component.

This page shows how to create agents by extending `bxModules.bxai.models.runnables.AiAgent`.

## Why Use Class-Based Agents?

Use class-based agents when you want:

* **Encapsulation**: Keep behavior, tools, and defaults in one class
* **Reusability**: Instantiate the same agent across modules and apps
* **Testability**: Unit-test agent setup methods independently
* **Consistency**: Standardize model params, guardrails, and instructions
* **Composition**: Build specialized agent families via inheritance

## Minimal Class-Based Agent

```javascript
// agents/SupportAgent.bx
class extends="bxModules.bxai.models.runnables.AiAgent" {

    function init() {
        super.init(
            name        : "SupportAgent",
            description : "Customer support specialist",
            instructions: "Resolve support issues clearly and politely"
        )

        return this
    }

}
```

Usage:

```javascript
agent = new agents.SupportAgent()
response = agent.run( "My order has not arrived yet" )
```

## Encapsulated Agent with Model, Memory, and Tools

```javascript
// agents/OrderSupportAgent.bx
class extends="bxModules.bxai.models.runnables.AiAgent" {

    function init() {
        super.init(
            name        : "OrderSupport",
            description : "Order support specialist",
            instructions: "Help users with order and shipping issues",
            model       : aiModel( provider: "openai", params: { model: "gpt-4o-mini" } ),
            memory      : aiMemory( "cache", { key: "order-support" } ),
            params      : { temperature: 0.2 }
        )

        registerTools()
        configurePolicies()

        return this
    }

    private function registerTools() {
        this.withTools( [
            aiTool(
                "getOrderStatus",
                "Get order status by order ID",
                ( orderId ) => orderService.getStatus( orderId )
            ),
            aiTool(
                "createEscalation",
                "Create escalation for unresolved issues",
                ( issueSummary ) => ticketService.createEscalation( issueSummary )
            )
        ] )
    }

    private function configurePolicies() {
        this.withInstructions( "Verify customer identity before sharing order details" )
            .withOptions( { returnFormat: "single" } )
    }

}
```

## Registering Class-Based Agents at Application Startup

This pattern keeps agent lifecycle management in one place:

```javascript
// Application.bx
class {

    function onApplicationStart() {
        aiAgentRegistry().register( new agents.OrderSupportAgent(), "my-app" )
        aiAgentRegistry().register( new agents.BillingAgent(), "my-app" )

        return true
    }

    function onApplicationEnd() {
        aiAgentRegistry().unregister( "OrderSupport@my-app" )
        aiAgentRegistry().unregister( "BillingAgent@my-app" )
    }

}
```

Resolve and run:

```javascript
supportAgent = aiAgentRegistry().get( "OrderSupport@my-app" )
result = supportAgent.run( "Where is order 12345?" )
```

## Base Class + Specialized Agents

You can define a common base agent and extend it:

```javascript
// agents/BaseCompanyAgent.bx
class extends="bxModules.bxai.models.runnables.AiAgent" {

    function init( required string name, required string description, required string instructions ) {
        super.init(
            name        : arguments.name,
            description : arguments.description,
            instructions: arguments.instructions,
            model       : aiModel( provider: "openai", params: { model: "gpt-4o-mini" } ),
            params      : { temperature: 0.2 },
            memory      : aiMemory( "window", { maxMessages: 20 } )
        )

        this.withOptions( { returnFormat: "single" } )

        return this
    }

}
```

```javascript
// agents/BillingAgent.bx
class extends="agents.BaseCompanyAgent" {

    function init() {
        super.init(
            name        : "BillingAgent",
            description : "Billing and invoice support",
            instructions: "Handle billing issues and explain invoices"
        )

        this.withTools( [
            aiTool( "lookupInvoice", "Find invoice details", invoiceId => billingService.lookup( invoiceId ) )
        ] )

        return this
    }

}
```

## Sub-Agents Inside an Encapsulated Agent

Class-based agents also work well for delegation:

```javascript
class extends="bxModules.bxai.models.runnables.AiAgent" {

    function init() {
        super.init(
            name        : "Coordinator",
            description : "Coordinates specialized agents",
            instructions: "Delegate to the best specialist when useful"
        )

        this.addSubAgent( new agents.ResearchAgent() )
            .addSubAgent( new agents.WriterAgent() )

        return this
    }

}
```

## Testing Pattern

Keep setup deterministic and verify config in tests:

```javascript
agent = new agents.OrderSupportAgent()
config = agent.getConfig()

expect( config.name ).toBe( "OrderSupport" )
expect( config.toolCount ).toBeGTE( 1 )
expect( config.description ).toInclude( "support" )
```

## When to Use `aiAgent()` vs Class-Based

| Approach | Best For |
|---|---|
| `aiAgent()` BIF | Quick scripts, prototypes, and inline composition |
| Class-based (`extends="...AiAgent"`) | Reusable domain agents, large apps, shared conventions |

## Related Pages

* [Getting Started](getting-started.md)
* [Sub-Agents & Hierarchy](hierarchy.md)
* [Tools & MCP](tools-and-mcp.md)
* [Advanced Patterns](advanced.md)
