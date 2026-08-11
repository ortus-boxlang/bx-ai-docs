---
description: Reference documentation for built-in functions in BoxLang AI module
icon: book
---

# Built-In Functions Reference

Complete reference documentation for all BoxLang AI built-in functions (BIFs). These functions provide the primary interface for AI operations in BoxLang.

## 📚 Overview

The BoxLang AI module provides 33 built-in functions organized into functional categories:

### 🗨️ Chat & Conversation

Core functions for AI chat interactions.

* [**`aiChat()`**](aichat.md) - Synchronous AI chat with simple interface
* [**`aiChatAsync()`**](aichatasync.md) - Asynchronous chat returning Future
* [**`aiChatStream()`**](aichatstream.md) - Streaming chat with real-time callbacks
* [**`aiChatRequest()`**](aichatrequest.md) - Create reusable request objects
* [**`aiWebSearch()`**](aiwebsearch.md) - Provider-agnostic web search with normalized results
* [**`aiWebSearchAsync()`**](aiwebsearchasync.md) - Asynchronous web search returning Future

### 🤖 Agents & Models

Create autonomous agents and model runnables.

* [**`aiAgent()`**](aiagent.md) - Create AI agents with tools, memory, and reasoning
* [**`aiAgentRegistry()`**](aiagentregistry.md) - Access the global agent registry singleton
* [**`aiModel()`**](aimodel.md) - Create AI model runnables for pipelines
* [**`aiService()`**](aiservice.md) - Get AI service provider instances

### 🖼️ Image Generation

Generate images from text prompts.

* [**`aiImage()`**](aiimage.md) - Generate images with direct or fluent builder APIs

### 💾 Memory & Context

Manage conversation history and knowledge bases.

* [**`aiMemory()`**](aimemory.md) - Create memory instances (conversation, vector, cache, etc.)

### 📄 Documents & RAG

Load and process documents for RAG workflows.

* [**`aiDocuments()`**](aidocuments.md) - Load documents with fluent API
* [**`aiChunk()`**](aichunk.md) - Chunk text into segments
* [**`aiEmbed()`**](aiembed.md) - Generate vector embeddings

### 🔄 Transformation & Pipelines

Transform data in AI pipelines.

* [**`aiMessage()`**](aimessage.md) - Build message structures with fluent API
* [**`aiTransform()`**](aitransform.md) - Create transformation runnables
* [**`aiPopulate()`**](aipopulate.md) - Populate classes from AI responses

### 🔧 Tools & Utilities

Extend AI capabilities and estimate costs.

* [**`aiTool()`**](aitool.md) - Create callable tools for agents
* [**`aiTokens()`**](aitokens.md) - Estimate token counts and costs

### 🔌 MCP (Model Context Protocol)

Connect AI to external tools and data sources.

* [**`MCP()`**](mcp.md) - Create MCP client for consuming servers
* [**`MCPServer()`**](mcpserver.md) - Create MCP server for exposing tools

### 🧑‍⚖️ Human-in-the-Loop & Gateways

Present approvals to a human and route them through a platform.

* [**`aiGateway()`**](aigateway.md) - Resolve a gateway (CLI, HTTP, or an external module) by name
* [**`gatewayRegistry()`**](gatewayregistry.md) - Registry external gateway modules register into
* [**`aiDecisionStore()`**](aidecisionstore.md) - Durable store for `approve_always`/`approve_session` grants

### 🛡️ Security

Defend against prompt injection and data leakage.

* [**`aiFence()`**](aifence.md) - Fence untrusted content so the model treats it as data, not instructions

## 🎯 Quick Reference

### Common Usage Patterns

#### Simple Chat

```javascript
response = aiChat( "What is BoxLang?" );
```

#### Agent with Tools

```javascript
agent = aiAgent(
    name: "Assistant",
    tools: [ myTool ],
    memory: aiMemory( "window" )
);
response = agent.run( "Help me with this task" );
```

#### RAG (Retrieval Augmented Generation)

```javascript
// Load documents into vector memory
vectorMemory = aiMemory( "chroma" );
aiDocuments( "/docs" ).toMemory( vectorMemory );

// Query with context
agent = aiAgent( memory: vectorMemory );
response = agent.run( "What does the documentation say about X?" );
```

#### Streaming Responses

```javascript
aiChatStream(
    "Tell me a long story",
    ( chunk ) => { write( chunk ); flush; }
);
```

#### Structured Output

```javascript
class Person {
    property name="firstName";
    property name="age" type="numeric";
}

person = aiChat(
    "Extract: John Doe, age 30",
    {},
    { returnFormat: new Person() }
);
```

## 📊 Function Categories by Use Case

### For Simple AI Calls

Start with these for basic AI interactions:

* `aiChat()` - Simplest sync chat
* `aiMessage()` - Build complex messages
* `aiService()` - Get provider instance

### For Long-Running Operations

Use async/streaming for better UX:

* `aiChatAsync()` - Non-blocking requests
* `aiChatStream()` - Real-time responses

### For Autonomous Behavior

Let AI reason and use tools:

* `aiAgent()` - Autonomous agents
* `aiTool()` - Create callable functions
* `aiMemory()` - Maintain context

### For Knowledge Bases (RAG)

Build AI that knows your data:

* `aiDocuments()` - Load documents
* `aiMemory()` - Vector storage
* `aiEmbed()` - Generate embeddings
* `aiChunk()` - Split documents

### For Pipelines

Chain AI operations:

* `aiModel()` - Model runnables
* `aiTransform()` - Data transformation
* `aiMessage()` - Fluent message building

### For External Integration

Connect AI to external systems:

* `MCP()` - Consume MCP servers
* `MCPServer()` - Expose tools via MCP
* `aiTool()` - Wrap any function

## 🔑 Key Concepts

### Return Formats

All chat functions support multiple return formats:

* **"single"**: Just the content string (default for `aiChat()`)
* **"all"**: Array of all messages
* **"raw"**: Complete API response with metadata
* **"json"**: Parsed JSON object
* **"xml"**: Parsed XML document
* **Class/Struct**: Structured output (populate target)

### Provider Selection

Three ways to specify AI provider:

1. **Default**: Uses module configuration
2. **Parameter**: `aiChat( msg, {}, { provider: "claude" } )`
3. **Environment**: Auto-detects `<PROVIDER>_API_KEY` variables

### Memory Types

Different memory for different needs:

* **Window**: Recent conversation (short-term)
* **Vector**: Semantic search (RAG, knowledge)
* **Cache**: Distributed storage (CacheBox)
* **File**: Simple persistence
* **JDBC**: Database-backed
* **Session**: User session scope

### Fluent APIs

Many functions return objects with chainable methods:

```javascript
docs = aiDocuments( "/path" )
    .recursive()
    .extensions( ["md"] )
    .chunkSize( 1000 )
    .filter( doc => doc.metadata.size < 100000 )
    .load();
```

## 🎓 Learning Path

### Beginner

1. Start with [`aiChat()`](aichat.md) for simple requests
2. Learn [`aiMessage()`](aimessage.md) for structured conversations
3. Try [`aiChatStream()`](aichatstream.md) for real-time responses

### Intermediate

4. Create [`aiAgent()`](aiagent.md) with basic tools
5. Use [`aiMemory()`](aimemory.md) for conversation context
6. Implement [`aiTool()`](aitool.md) for custom functions

### Advanced

7. Build RAG systems with [`aiDocuments()`](aidocuments.md) and vector memory
8. Use [`aiChatAsync()`](aichatasync.md) for concurrent requests
9. Create MCP servers with [`MCPServer()`](mcpserver.md)
10. Build complex pipelines with [`aiModel()`](aimodel.md) and [`aiTransform()`](aitransform.md)

## 🔍 Function Index

| Function                              | Category  | Description                 |
| ------------------------------------- | --------- | --------------------------- |
| [`aiAgent()`](aiagent.md)             | Agents    | Create autonomous AI agents |
| [`aiChat()`](aichat.md)               | Chat      | Synchronous AI chat         |
| [`aiChatAsync()`](aichatasync.md)     | Chat      | Asynchronous AI chat        |
| [`aiChatRequest()`](aichatrequest.md) | Chat      | Create request objects      |
| [`aiChatStream()`](aichatstream.md)   | Chat      | Streaming AI chat           |
| [`aiChunk()`](aichunk.md)             | Documents | Chunk text into segments    |
| [`aiDecisionStore()`](aidecisionstore.md) | HITL  | Durable approval-grant store |
| [`aiDocuments()`](aidocuments.md)     | Documents | Load documents for RAG      |
| [`aiEmbed()`](aiembed.md)             | Documents | Generate embeddings         |
| [`aiFence()`](aifence.md)             | Security  | Fence untrusted content      |
| [`aiGateway()`](aigateway.md)         | Gateways  | Resolve a gateway by name    |
| [`aiGlobalSkills()`](aiglobalskills.md) | Skills  | List globally loaded skills |
| [`aiImage()`](aiimage.md)             | Image     | Generate images             |
| [`aiMemory()`](aimemory.md)           | Memory    | Create memory instances     |
| [`aiMessage()`](aimessage.md)         | Messages  | Build message structures    |
| [`aiModel()`](aimodel.md)             | Models    | Create model runnables      |
| [`aiParallel()`](aiparallel.md)       | Pipelines | Run branches in parallel    |
| [`aiAgentRegistry()`](aiagentregistry.md) | Agents | Global agent registry    |
| [`aiPopulate()`](aipopulate.md)       | Utilities | Populate classes from JSON  |
| [`aiService()`](aiservice.md)         | Services  | Get service providers       |
| [`aiSkill()`](aiskill.md)             | Skills    | Create a skill definition   |
| [`aiSpeak()`](aispeak.md)             | Audio     | Text-to-speech generation   |
| [`aiTokens()`](aitokens.md)           | Utilities | Estimate token counts       |
| [`aiTool()`](aitool.md)               | Tools     | Create callable tools       |
| [`aiToolRegistry()`](aitoolregistry.md) | Tools   | Access the global tool registry |
| [`aiTranscribe()`](aitranscribe.md)   | Audio     | Speech-to-text transcription |
| [`aiTransform()`](aitransform.md)     | Transform | Create transformers         |
| [`aiTranslate()`](aitranslate.md)     | Audio     | Audio-to-English translation |
| [`aiWebSearch()`](aiwebsearch.md)       | Search    | Search web content          |
| [`aiWebSearchAsync()`](aiwebsearchasync.md) | Search  | Async web search            |
| [`gatewayRegistry()`](gatewayregistry.md) | Gateways | Registry for external gateway modules |
| [`MCP()`](mcp.md)                     | MCP       | Create MCP client           |
| [`MCPServer()`](mcpserver.md)         | MCP       | Create MCP server           |

## 📖 Additional Resources

* [**Getting Started Guide**](../../../getting-started/getting-started.md) - Introduction to BoxLang AI
* [**Agents Documentation**](../../../main-components/agents/) - Deep dive into agents
* [**Memory Systems**](../../../main-components/memory/) - Memory types and usage
* [**RAG Guide**](../../../rag/rag.md) - Build knowledge-based AI
* [**Transformers**](../../../main-components/transformers/README.md) - Data transformation
* [**Examples**](https://github.com/ortus-boxlang/bx-ai/tree/development/examples) - Working code examples

## 💡 Tips

* **Start simple**: Begin with `aiChat()` before moving to agents
* **Use appropriate memory**: Window for chat, vector for knowledge
* **Clear tool descriptions**: Help AI choose correct tools
* **Handle errors**: Wrap AI calls in try/catch blocks
* **Monitor costs**: Use `aiTokens()` to estimate usage
* **Test locally**: Use Ollama for free local testing
* **Stream long responses**: Better UX with `aiChatStream()`
* **Async for parallel**: Use `aiChatAsync()` for multiple concurrent requests
