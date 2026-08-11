---
description: >-
  Recommended practices for embedding models, metadata filtering, performance,
  and advanced vector memory usage patterns.
icon: list-check
---

# Best Practices & Advanced Usage

## Best Practices

### 1. Choose Appropriate Embedding Models

```java
// Small, fast, cost-effective
embeddingModel: "text-embedding-3-small"    // OpenAI - 1536 dimensions

// Large, more accurate
embeddingModel: "text-embedding-3-large"    // OpenAI - 3072 dimensions

// Local, free
embeddingModel: "nomic-embed-text"          // Ollama - 768 dimensions
```

### 2. Use Metadata for Filtering

```java
// Add metadata when storing
memory.add({
    text: "User reported billing issue",
    metadata: {
        userId: "user123",
        category: "billing",
        priority: "high",
        timestamp: now()
    }
})

// Filter on retrieval
relevant = memory.getRelevant(
    query: "payment problems",
    limit: 5,
    filter: { category: "billing", priority: "high" }
)
```

### 3. Optimize Collection Size

```java
// Periodic cleanup of old vectors
function cleanupOldVectors( memory, daysOld = 90 ) {
    var cutoffDate = dateAdd( "d", -daysOld, now() );
    memory.deleteByFilter({
        timestamp_lt: cutoffDate
    })
}
```

### 4. Monitor Performance

```java
// Enable logging for debugging
memory = aiMemory( memory: "pinecone", config: {
    collection: "prod",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    cache: true,
    logRequests: true,          // Log vector operations
    logRequestToConsole: false
} )
```

### 5. Use Hybrid for User-Facing Apps

```java
// Combines recent conversation with relevant history
memory = aiMemory( memory: "hybrid", config: {
    recentLimit: 5,              // Always include last 5 messages
    semanticLimit: 3,            // Add 3 relevant past messages
    totalLimit: 8,               // Max 8 total
    vectorProvider: "qdrant"
} )
```

### 6. Dimension Matching

Ensure embedding dimensions match across your application:

```java
// OpenAI text-embedding-3-small = 1536 dimensions
memory = aiMemory( memory: "pinecone", config: {
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    dimensions: 1536                // Must match model output
} )
```

### 7. Use Multi-Tenant Isolation

Securely isolate user and conversation data in shared collections:

```java
// Enterprise multi-user application
function getUserMemory( userId, conversationId = "" ) {
    return aiMemory( memory: "postgres",
        key: createUUID(),
        userId: arguments.userId,
        conversationId: arguments.conversationId,
        config: {
            collection: "enterprise_vectors",
            datasource: "mainDB",
            embeddingProvider: "openai"
        }
    )
}

// Automatic isolation - no manual filtering needed
aliceMemory = getUserMemory( "alice", "support-123" )
bobMemory = getUserMemory( "bob", "sales-456" )

// Each user only sees their own vectors
aliceResults = aliceMemory.getRelevant( "billing", 5 )  // Only Alice's data
bobResults = bobMemory.getRelevant( "billing", 5 )      // Only Bob's data
```

***

## Advanced Usage

### Custom Similarity Thresholds

```java
// Only retrieve highly relevant messages
relevant = memory.getRelevant(
    query: userInput,
    limit: 10,
    minScore: 0.8               // 80% similarity threshold
)
```

### Multi-Collection Strategy

```java
// Separate collections for different contexts
supportMemory = aiMemory( memory: "pinecone", config: {
    collection: "customer_support",
    embeddingProvider: "openai"
} )

salesMemory = aiMemory( memory: "pinecone", config: {
    collection: "sales_conversations",
    embeddingProvider: "openai"
} )

// Use appropriate memory based on context
memory = userType == "support" ? supportMemory : salesMemory
```

### Cross-Session Continuity

```java
// Per-user persistent memory with multi-tenant isolation
function getUserMemory( userId ) {
    return aiMemory( memory: "postgres",
        key: createUUID(),
        userId: arguments.userId,
        config: {
            collection: "user_history",
            embeddingProvider: "openai",
            embeddingModel: "text-embedding-3-small",
            datasource: "mainDB"
        }
    )
}

// Each user's data is automatically isolated
userMemory = getUserMemory( session.userId )
agent = aiAgent( name: "Assistant", memory: userMemory )

// Conversations persist across sessions
agent.run( "What did we discuss last week?" )  // Retrieves user's history only
```

### Batch Operations

```java
// Add multiple messages efficiently
messages.each( function( msg ) {
    memory.add( msg )
})

// Better: use batch add (if supported by provider)
memory.addBatch( messages )
```

