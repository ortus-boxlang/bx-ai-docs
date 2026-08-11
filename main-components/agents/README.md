---
description: >-
  AI Agents are autonomous entities that reason, use tools, and maintain
  conversation memory to handle complex AI workflows.
icon: robot
---

# AI Agents

AI Agents are autonomous entities that can reason, use tools, and maintain conversation memory. Inspired by LangChain agents but "Boxified" for simplicity and productivity, agents handle complex AI workflows by automatically managing state, context, and tool execution.

## 🎯 What are AI Agents?

An agent is more than a simple chat interface — it's an intelligent entity that:

* **Maintains Memory** — Remembers conversation history across interactions
* **Uses Tools** — Can call functions to access data, perform calculations, or interact with systems
* **Applies Skills** — Injects domain knowledge into the system context (v3.0+)
* **Runs Middleware** — Intercepts lifecycle events for logging, retry, and guardrails (v3.0+)
* **Reasons and Plans** — Determines when and how to use tools to accomplish tasks
* **Delegates to Sub-Agents** — Can orchestrate specialized sub-agents for complex tasks
* **Integrates with Pipelines** — Works seamlessly as a stage in BoxLang AI pipelines

## 🏗️ Agent Architecture

```mermaid
graph TB
    subgraph "Agent Components"
        A[🤖 Agent Core]
        M[🧠 AI Model]
        MEM[💭 Memory System]
        T[🛠️ Tools]
        SK[📚 Skills]
        MW[⚙️ Middleware]
    end

    subgraph "Memory Types"
        W[Window Memory]
        S[Summary Memory]
        V[Vector Memory]
        F[File Memory]
    end

    subgraph "External Systems"
        API[External APIs]
        DB[Databases]
        MCP[MCP Servers]
    end

    A --> M
    A --> MEM
    A --> T
    A --> SK
    A --> MW

    MEM --> W
    MEM --> S
    MEM --> V
    MEM --> F

    T --> API
    T --> DB
    T --> MCP

    style A fill:#BD10E0
    style M fill:#4A90E2
    style MEM fill:#50E3C2
    style T fill:#B8E986
    style SK fill:#F5A623
    style MW fill:#D0021B
```

## 🔄 Agent Decision Flow

```mermaid
sequenceDiagram
    participant U as User
    participant MW as Middleware
    participant A as Agent
    participant MEM as Memory
    participant AI as AI Model
    participant T as Tools/Skills

    U->>MW: User input
    MW->>A: beforeAgentRun hook
    A->>MEM: Retrieve context
    MEM->>A: History + skill context
    A->>AI: Messages + tools + skills

    alt AI needs tool
        AI->>A: Tool call request
        MW->>A: beforeToolCall hook
        A->>T: Execute tool
        T->>A: Tool result
        MW->>A: afterToolCall hook
        A->>AI: Send tool result
        AI->>A: Final response
    else AI has answer
        AI->>A: Direct response
    end

    A->>MEM: Store new messages
    MW->>A: afterAgentRun hook
    A->>U: Return response
```

## 📖 Sub-Pages

| Page | Description |
|---|---|
| [Getting Started](getting-started.md) | Creating agents, configuration, model selection, return formats |
| [Class-Based Agents](class-based-agents.md) | Build encapsulated reusable agents by extending `AiAgent` |
| [Memory Management](memory.md) | Memory types, per-call identity routing, resume/suspend, multi-tenant |
| [Tools & MCP](tools-and-mcp.md) | Tool Registry, ClosureTool, MCP server seeding |
| [Skills](skills.md) | Always-on and lazy-loaded skills for domain knowledge injection |
| [Middleware](middleware.md) | Lifecycle hooks for logging, retry, guardrails, human-in-the-loop |
| [Sub-Agents & Hierarchy](hierarchy.md) | Parent-child agent delegation patterns |
| [Streaming](streaming.md) | Real-time streaming responses |
| [RAG & Document Loading](rag.md) | Knowledge base integration and document retrieval |
| [Transformers](transformers.md) | Input/output transformation and structured data extraction |
| [Advanced Patterns](advanced.md) | Pipelines, event interception, best practices |

## Quick Start

```javascript
// Minimal agent
agent = aiAgent(
    name: "Assistant",
    description: "A helpful AI assistant",
    instructions: "Be concise and friendly"
)

response = agent.run( "What is BoxLang?" )
println( response )
```

```javascript
// Full-featured agent (v3.0)
agent = aiAgent(
    name           : "SupportBot",
    description    : "Customer support specialist",
    instructions   : "Help customers with product questions",
    model          : aiModel( "openai" ),
    tools          : [ searchTool, ticketTool ],
    memory         : aiMemory( "cache" ),
    skills         : aiSkill( ".ai/skills" ),
    availableSkills: aiSkill( ".ai/advanced-skills" ),
    middleware     : [ new LoggingMiddleware(), new RetryMiddleware() ],
    mcpServers     : [ { url: "http://tools-server/mcp", toolNames: ["search"] } ]
)

response = agent.run( "My order hasn't arrived" )
```

## Related Documentation

* [AI Skills](../skills.md) — Full skills reference
* [Tool Registry](../tool-registry.md) — Global tool registry
* [Middleware](../middleware.md) — Full middleware reference
* [Human-in-the-Loop](../human-in-the-loop.md) — Suspend an agent for approval before sensitive tool calls
* [Memory Systems](../memory/README.md) — Memory configuration
* [Pipelines](../pipelines/README.md) — Using agents in pipelines
* [Security Guide](../../deployment/security.md) — Guardrails against prompt injection and data leakage
