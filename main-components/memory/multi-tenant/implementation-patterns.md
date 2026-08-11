---
description: >-
  Implementation patterns, memory type strategies, and vector memory
  multi-tenancy for isolated conversations.
icon: diagram-project
---

# Implementation Patterns

## 🔧 Implementation Patterns

### Pattern 1: Single Conversation Per User

Simplest pattern - one conversation per user:

```java
class {
    property name="userMemories" default="{}";

    function getUserMemory( required string userId ) {
        if ( !variables.userMemories.keyExists( arguments.userId ) ) {
            variables.userMemories[ arguments.userId ] = aiMemory( memory: "session",
                key: "chatbot",
                userId: arguments.userId,
                config: { maxMessages: 20 }
            );
        }
        return variables.userMemories[ arguments.userId ];
    }

    function chat( required string userId, required string message ) {
        var memory = getUserMemory( arguments.userId );
        var agent = aiAgent( name: "Assistant", memory: memory );
        return agent.run( arguments.message );
    }
}
```

### Pattern 2: Multiple Conversations Per User

Enterprise pattern - users have multiple concurrent conversations:

```java
class {
    function getConversationMemory(
        required string userId,
        required string conversationId
    ) {
        return aiMemory( memory: "cache",
            key: "chat",
            userId: arguments.userId,
            conversationId: arguments.conversationId,
            config: {
                cacheName: "default",
                maxMessages: 30,
                cacheTimeout: 3600
            }
        );
    }

    function sendMessage(
        required string userId,
        required string conversationId,
        required string message
    ) {
        var memory = getConversationMemory(
            arguments.userId,
            arguments.conversationId
        );

        var agent = aiAgent(
            name: "Support Bot",
            memory: memory
        );

        return agent.run( arguments.message );
    }

    function listConversations( required string userId ) {
        // Query database for user's conversations
        return queryExecute(
            "SELECT conversation_id, created_at, last_message
             FROM conversations
             WHERE user_id = :userId
             ORDER BY last_message DESC",
            { userId: arguments.userId }
        );
    }
}
```

### Pattern 3: Hierarchical Isolation (Organization → User → Conversation)

Complex enterprise pattern with organization-level isolation:

```java
class {
    function getMemory(
        required string organizationId,
        required string userId,
        required string conversationId
    ) {
        // Use organization ID as prefix for complete isolation
        var compositeUserId = "#arguments.organizationId#:#arguments.userId#";

        return aiMemory( memory: "jdbc",
            key: createUUID(),
            userId: compositeUserId,
            conversationId: arguments.conversationId,
            config: {
                datasource: "mainDB",
                table: "ai_conversations",
                maxMessages: 100
            }
        );
    }
}
```

### Pattern 4: Role-Based Memory Switching

Different memory types based on user roles:

```java
function getRoleBasedMemory( required struct user ) {
    var memoryType = "window";  // Default
    var maxMessages = 10;

    // Premium users get better memory
    if ( arguments.user.role == "premium" ) {
        memoryType = "summary";
        maxMessages = 50;
    }
    // Enterprise users get vector memory
    else if ( arguments.user.role == "enterprise" ) {
        memoryType = "chroma";
        maxMessages = 0;  // Unlimited
    }

    return aiMemory( memoryType,
        key: createUUID(),
        userId: arguments.user.id,
        config: {
            maxMessages: maxMessages,
            summaryThreshold: 20,  // For summary memory
            collection: "enterprise_vectors",  // For vector memory
            embeddingProvider: "openai"
        }
    );
}
```

***


## Memory Type Strategies

### Standard Memory Multi-Tenancy

All standard memory types support multi-tenant isolation:

#### Windowed Memory

```java
// Shared infrastructure, isolated per user
memory = aiMemory( memory: "window",
    key: createUUID(),
    userId: session.userId,
    conversationId: url.chatId,
    config: { maxMessages: 10 }
)
```

**Isolation method**: In-memory arrays per userId/conversationId combination

#### Summary Memory

```java
// Long conversations with summarization
memory = aiMemory( memory: "summary",
    key: createUUID(),
    userId: session.userId,
    conversationId: "support",
    config: {
        maxMessages: 30,
        summaryThreshold: 15,
        summaryModel: "gpt-4o-mini"
    }
)
```

**Isolation method**: Summaries stored per userId/conversationId

#### Session Memory

```java
// Automatic session-based isolation
memory = aiMemory( memory: "session",
    key: "chatbot",
    userId: session.userId,
    conversationId: request.conversationId,
    config: { maxMessages: 20 }
)
```

**Isolation method**: Composite session key (key + userId + conversationId)

#### File Memory

```java
// File-based isolation with automatic naming
memory = aiMemory( memory: "file",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        directoryPath: "/app/memories",
        maxMessages: 50
    }
)
// Creates: /app/memories/[key]_user123_chat456.json
```

**Isolation method**: Separate JSON file per userId/conversationId

#### Cache Memory

```java
// Distributed cache with isolation
memory = aiMemory( memory: "cache",
    key: "chat",
    userId: session.userId,
    conversationId: url.conversationId,
    config: {
        cacheName: "redis",
        cacheTimeout: 3600,
        maxMessages: 30
    }
)
```

**Isolation method**: Composite cache key (key + userId + conversationId)

#### JDBC Memory

```java
// Database with userId/conversationId columns
memory = aiMemory( memory: "jdbc",
    key: createUUID(),
    userId: session.userId,
    conversationId: request.ticketId,
    config: {
        datasource: "mainDB",
        table: "ai_conversations",
        maxMessages: 100,
        autoCreate: true
    }
)
```

**Isolation method**: Database columns with WHERE filtering

**Table structure:**

```sql
CREATE TABLE ai_conversations (
    id VARCHAR(50) PRIMARY KEY,
    user_id VARCHAR(100),           -- Multi-tenant support
    conversation_id VARCHAR(100),   -- Multi-conversation support
    role VARCHAR(20),
    content TEXT,
    metadata TEXT,
    created_at TIMESTAMP,
    INDEX idx_user (user_id),
    INDEX idx_conversation (conversation_id),
    INDEX idx_tenant (user_id, conversation_id)  -- Composite index
);
```

***


## Vector Memory Multi-Tenancy

All 12 vector memory providers support multi-tenant isolation:

### BoxVector (In-Memory)

```java
memory = aiMemory( memory: "boxvector",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "dev_vectors",
        embeddingProvider: "openai"
    }
)
```

**Storage**: Metadata with in-memory filtering

### Chroma

```java
memory = aiMemory( memory: "chroma",
    key: createUUID(),
    userId: "user123",
    conversationId: "support",
    config: {
        collection: "customer_support",
        embeddingProvider: "openai",
        host: "localhost",
        port: 8000
    }
)
```

**Storage**: Metadata with $and operator filtering

### PostgreSQL (pgvector)

```java
memory = aiMemory( memory: "postgres",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "ai_vectors",
        datasource: "mainDB",
        embeddingProvider: "openai",
        dimensions: 1536
    }
)
```

**Storage**: Dedicated VARCHAR(255) columns with composite index

### MySQL (9+ Native Vectors)

```java
memory = aiMemory( memory: "mysql",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "ai_vectors",
        datasource: "mainDB",
        embeddingProvider: "openai",
        dimensions: 1536
    }
)
```

**Storage**: Dedicated VARCHAR(255) columns with composite index

### TypeSense

```java
memory = aiMemory( memory: "typesense",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "fast_search",
        host: "localhost",
        port: 8108,
        apiKey: "xyz",
        embeddingProvider: "openai"
    }
)
```

**Storage**: Root-level fields with := filter syntax

### Pinecone

```java
memory = aiMemory( memory: "pinecone",
    key: createUUID(),
    userId: "user123",
    conversationId: "prod",
    config: {
        collection: "production_vectors",
        apiKey: getSystemSetting( "PINECONE_API_KEY" ),
        environment: "us-west1-gcp",
        embeddingProvider: "openai"
    }
)
```

**Storage**: Metadata with $eq operators

### Qdrant

```java
memory = aiMemory( memory: "qdrant",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "high_perf_vectors",
        host: "localhost",
        port: 6333,
        embeddingProvider: "openai"
    }
)
```

**Storage**: Payload root-level fields with match filters

### Weaviate

```java
memory = aiMemory( memory: "weaviate",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "KnowledgeVectors",  // PascalCase
        host: "localhost",
        port: 8080,
        embeddingProvider: "openai"
    }
)
```

**Storage**: Properties root-level with GraphQL Equal operator

### Milvus

```java
memory = aiMemory( memory: "milvus",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "enterprise_vectors",
        host: "localhost",
        port: 19530,
        embeddingProvider: "openai"
    }
)
```

**Storage**: Metadata with filter expressions

### Hybrid Memory

```java
memory = aiMemory( memory: "hybrid",
    key: createUUID(),
    userId: "user123",
    conversationId: "support",
    config: {
        recentLimit: 5,
        semanticLimit: 5,
        vectorProvider: "chroma",
        vectorConfig: {
            collection: "hybrid_vectors",
            embeddingProvider: "openai"
        }
    }
)
```

**Storage**: Delegates to underlying vector provider (Chroma in this example)

***

