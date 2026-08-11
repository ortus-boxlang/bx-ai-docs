---
description: >-
  Advanced agent patterns: pipeline integration, dynamic tools, introspection,
  event interception, and best practices.
icon: gears
---

# Advanced Patterns

## Pipeline Integration

Agents implement `IAiRunnable`, so they plug directly into composable pipelines:

```javascript
agent = aiAgent(
    name        : "Summarizer",
    instructions: "Create concise summaries of the provided content"
)

pipeline = aiMessage()
    .user( "Task: ${task}" )
    .to( agent )
    .transform( r => r.toUpper() )

result = pipeline.run( { task: "Summarize AI trends in 2025" } )
```

### Chaining Agents

Multiple agents can be chained in sequence, each processing the output of the previous:

```javascript
researchAgent = aiAgent( name: "Researcher",  instructions: "Research topics thoroughly" )
summaryAgent  = aiAgent( name: "Summarizer",  instructions: "Create concise summaries" )
editorAgent   = aiAgent( name: "Editor",      instructions: "Polish and format content" )

pipeline = aiMessage()
    .user( "Research: ${topic}" )
    .to( researchAgent )
    .transform( r => "Summarize this: ${r}" )
    .to( summaryAgent )
    .transform( r => "Edit and polish: ${r}" )
    .to( editorAgent )

result = pipeline.run( { topic: "Quantum Computing" } )
```

## Dynamic Tool Assignment

Assign different tools to an agent based on runtime context:

```javascript
function getToolsForRole( required string userRole ) {
    if ( userRole == "admin" ) {
        return [ adminTool, userTool, reportTool ]
    }
    return [ userTool ]
}

agent = aiAgent( name: "ContextAgent" )
    .setTools( getToolsForRole( getCurrentUserRole() ) )
```

## Agent Introspection

Inspect a running agent's full configuration at any time via `getConfig()`:

```javascript
agent = aiAgent(
    name        : "Inspector",
    description : "Analysis agent",
    instructions: "Analyze data carefully",
    model       : aiModel( "openai", { temperature: 0.7 } ),
    tools       : [ searchTool, calculatorTool ],
    params      : { maxTokens: 2000 }
)

config = agent.getConfig()

// Core identity
println( config.name )         // "Inspector"
println( config.description )  // "Analysis agent"

// Model details
println( config.model.name )               // "gpt-4o-mini"
println( config.model.provider )           // "openai"
println( config.model.toolCount )          // 2
println( config.model.params.temperature ) // 0.7

// Memory info (array of memory summaries)
config.memories.each( function( mem ) {
    println( mem.type )         // e.g., "SessionMemory"
    println( mem.messageCount ) // Number of stored messages
} )

// Execution parameters
println( config.params.maxTokens )       // 2000
println( config.options.returnFormat )   // "single" (default)

// 3.0 properties
println( config.activeSkillCount )    // Number of always-on skills loaded
println( config.availableSkillCount ) // Number of lazy skills in pool
```

## Conditional Agent Execution

Create different agents based on runtime conditions:

```javascript
function processRequest( required string userInput, required boolean requiresTools ) {
    if ( requiresTools ) {
        agent = aiAgent(
            name : "ToolAgent",
            tools: [ weatherTool, calculatorTool ]
        )
    } else {
        agent = aiAgent( name: "SimpleAgent" )
    }

    return agent.run( userInput )
}
```

## Event Interception

Agents fire events at key lifecycle points. Use `BoxRegisterInterceptor()` to observe them:

```javascript
interceptor = {
    beforeAIAgentRun: function( data ) {
        writeLog( "Agent '#data.agent.getName()#' starting with: #data.input#" )
    },
    afterAIAgentRun: function( data ) {
        writeLog( "Agent completed. Response length: #len( data.response )#" )
    },
    beforeAIToolCall: function( data ) {
        writeLog( "Calling tool: #data.toolName# with args: #jsonSerialize( data.args )#" )
    },
    afterAIToolCall: function( data ) {
        writeLog( "Tool #data.toolName# returned: #data.result#" )
    }
}

BoxRegisterInterceptor( interceptor )

// Run agent — all registered events will fire
agent.run( "What is the weather in Paris?" )
```

See [Events Reference](../../advanced/events/README.md) for the full list of agent events.

## 5 Best Practices

### 1. Provide Clear, Specific Instructions

```javascript
// Good: Specific role + behavior expectations
agent = aiAgent(
    name        : "SupportAgent",
    description : "Customer support specialist",
    instructions: """
        You are a friendly customer support agent.
        - Always be polite and professional
        - Ask clarifying questions when needed
        - Provide step-by-step solutions
        - Use tools to look up order information
        - Escalate complex issues with the escalate tool
    """
)
```

### 2. Give Agents Only the Tools They Need

```javascript
// Only wire up relevant tools — fewer tools = better decisions
agent = aiAgent(
    name : "WeatherAgent",
    tools: [ weatherTool ]  // Don't add unrelated tools
)
```

### 3. Manage Memory Lifecycle

```javascript
agent = aiAgent( name: "SessionAgent", memory: aiMemory( "window" ) )

agent.run( "Help me with task X" )

// Clear memory when a session ends or a new topic begins
agent.clearMemory()
```

### 4. Tune Parameters per Task Type

```javascript
// Creative tasks: higher temperature + more tokens
creativeAgent = aiAgent(
    name  : "Writer",
    params: { temperature: 0.8, max_tokens: 1000 }
)

// Factual/analytical tasks: lower temperature
factualAgent = aiAgent(
    name  : "Analyzer",
    params: { temperature: 0.2, max_tokens: 500 }
)
```

### 5. Handle Errors Gracefully

```javascript
try {
    response = agent.run( userInput )
} catch ( any e ) {
    writeLog( "Agent error: #e.message#", "error" )
    // Route to fallback logic
    response = "I encountered an error. Please try again."
}
```

## Related Pages

* [Getting Started](getting-started.md) — Creating and configuring agents
* [Hierarchy & Sub-Agents](hierarchy.md) — Delegating between agents
* [Streaming](streaming.md) — Streaming with agents
* [Transformers](transformers.md) — Processing inputs and outputs
* [Events Reference](../../advanced/events/README.md) — Full event catalog
