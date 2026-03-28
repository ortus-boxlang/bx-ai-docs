---
description: >-
  Sub-agents allow parent agents to delegate tasks to specialized child agents,
  enabling complex multi-agent orchestration patterns.
icon: sitemap
---

# Sub-Agents & Hierarchy

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

Sub-agents allow you to create specialized agents that a parent agent can delegate to. When you register a sub-agent, it is automatically wrapped as an internal tool that the parent can invoke by calling `delegate_to_{agent_name}`.

## Creating Agents with Sub-Agents

```javascript
// Create specialized sub-agents
mathAgent = aiAgent(
    name        : "MathAgent",
    description : "A mathematics expert",
    instructions: "You help with mathematical calculations and concepts"
)

codeAgent = aiAgent(
    name        : "CodeAgent",
    description : "A programming expert",
    instructions: "You help with code review, writing, and debugging"
)

// Create parent agent with sub-agents
mainAgent = aiAgent(
    name        : "OrchestratorAgent",
    description : "Main coordinator that delegates to specialists",
    instructions: """
        Analyze each request and delegate to appropriate sub-agents:
        - MathAgent: For mathematical tasks
        - CodeAgent: For programming tasks
        Answer directly for simple queries that don't require specialists.
    """,
    subAgents   : [ mathAgent, codeAgent ]
)

// The parent agent automatically has delegation tools available
response = mainAgent.run( "Write a function to calculate factorial" )
// → OrchestratorAgent decides to delegate to CodeAgent
```

## Fluent Sub-Agent API

```javascript
mainAgent = aiAgent( name: "MainAgent" )
    .addSubAgent( helperAgent )
    .addSubAgent( specialistAgent )

// Or replace all sub-agents at once
mainAgent.setSubAgents( [ newAgent1, newAgent2 ] )
```

## Sub-Agent Management

```javascript
// Check if a sub-agent exists
if ( mainAgent.hasSubAgent( "MathAgent" ) ) {
    println( "Math agent is available" )
}

// Get a specific sub-agent
mathAgent = mainAgent.getSubAgent( "MathAgent" )
if ( !isNull( mathAgent ) ) {
    result = mathAgent.run( "What is 2 + 2?" )
}

// Get all sub-agents
allSubAgents = mainAgent.getSubAgents()
println( "Total sub-agents: #allSubAgents.len()#" )
```

## Sub-Agent Config Introspection

```javascript
config = mainAgent.getConfig()

println( config.subAgentCount )  // Number of sub-agents

config.subAgents.each( agent => {
    println( "Name: #agent.name#, Desc: #agent.description#" )
} )
```

## How Delegation Works

When a sub-agent is added, it is wrapped as a tool automatically:

* **Tool name**: `delegate_to_{agent_name}` (lowercase, special characters replaced with underscores)
* **Tool description**: Built from the sub-agent's `name` and `description`
* **Tool parameter**: A single `task` parameter for the query to delegate

The parent agent's AI model decides when to use the delegation tool based on task context.

```javascript
// Behind the scenes, adding a sub-agent creates a tool like:
// Tool name: "delegate_to_mathagent"
// Tool description: "Delegate a task to the 'MathAgent' sub-agent: A mathematics expert"
// Tool call: subAgent.run( task )
```

## Three-Level Hierarchy Example

```javascript
// Leaf agents — specialized experts
sqlAgent = aiAgent(
    name        : "SQLExpert",
    description : "Writes and optimizes SQL queries",
    instructions: "Generate efficient, indexed SQL queries"
)

pyAgent  = aiAgent(
    name        : "PythonExpert",
    description : "Writes and reviews Python code",
    instructions: "Write clean, Pythonic code with type hints"
)

// Mid-level agent — engineering coordinator
engineeringAgent = aiAgent(
    name      : "EngineeringLead",
    description: "Coordinates engineering tasks",
    subAgents : [ sqlAgent, pyAgent ]
)

// Top-level orchestrator
cto = aiAgent(
    name      : "CTO",
    description: "Strategic technical director",
    subAgents : [ engineeringAgent ]
)

// CTO can cascade delegation down to leaf agents
response = cto.run( "Optimize our order search query" )
// CTO → EngineeringLead → SQLExpert
```

## Sub-Agents vs. Tools

| | Sub-Agents | Tools |
|---|---|---|
| **Best for** | Delegating reasoning tasks to another AI | Calling external systems/APIs |
| **Invokes** | Another AI model | A function/lambda |
| **Context** | Child agent has its own memory/instructions | Single function call |
| **Use when** | Task requires dedicated expertise or instructions | Task requires data/computation |

## Sub-Agents with Memory

Each sub-agent maintains its own memory independently:

```javascript
researchAgent = aiAgent(
    name  : "Researcher",
    memory: aiMemory( "window" )  // Independent memory
)

writerAgent = aiAgent(
    name  : "Writer",
    memory: aiMemory( "window" )  // Independent memory
)

coordinator = aiAgent(
    name     : "Coordinator",
    subAgents: [ researchAgent, writerAgent ]
)
```

## Related Pages

* [Getting Started](getting-started.md) — Creating basic agents
* [Tools & MCP](tools-and-mcp.md) — Tools vs. sub-agents
* [Advanced Patterns](advanced.md) — Pipeline-based agent chaining
