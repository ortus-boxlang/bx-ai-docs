---
description: >-
  Combine recent conversation history with semantic search for balanced,
  cost-effective context retrieval.
icon: shuffle
---

# Hybrid Memory

**HybridMemory** combines the benefits of both standard memory (recency) and vector memory (relevance).

### How It Works

1. Maintains recent messages in a window
2. Stores all messages in vector database
3. Returns combination of recent + semantically relevant messages
4. Automatically deduplicates

### Configuration

```java
memory = aiMemory( memory: "hybrid", config: {
    recentLimit: 5,                 // Number of recent messages
    semanticLimit: 5,               // Number of semantic matches
    totalLimit: 10,                 // Max combined messages
    recentWeight: 0.6,              // 60% recent, 40% semantic
    vectorProvider: "chroma",       // Vector backend
    vectorConfig: {                 // Vector provider config
        collection: "hybrid_chat",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small"
    }
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user/conversation isolation in hybrid memory
memory = aiMemory( memory: "hybrid",
    key: createUUID(),
    userId: "alice",
    conversationId: "support-chat",
    config: {
        recentLimit: 5,
        semanticLimit: 5,
        vectorProvider: "chroma",
        vectorConfig: {
            collection: "hybrid_chat",
            embeddingProvider: "openai"
        }
    }
)
```

### Benefits

* **Recent Context**: Always includes latest messages
* **Semantic Relevance**: Finds related past conversations
* **Balanced**: Best of both approaches
* **Automatic**: No manual context management

### Use Cases

```java
// Customer support with history
agent = aiAgent(
    name: "Support Agent",
    memory: aiMemory( memory: "hybrid", config: {
        recentLimit: 3,             // Last 3 messages
        semanticLimit: 5,           // 5 relevant past cases
        vectorProvider: "pinecone",
        vectorConfig: {
            collection: "support_history",
            embeddingProvider: "openai"
        }
    } )
)

// Agent automatically gets recent conversation + relevant past cases
agent.run( "I'm having the same billing issue as before" )
```

