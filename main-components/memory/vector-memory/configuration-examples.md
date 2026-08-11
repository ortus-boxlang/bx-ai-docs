---
description: >-
  Ready-to-use vector memory configuration examples for development,
  production cloud, and production self-hosted setups.
icon: gear
---

# Configuration Examples

### Development Setup

```java
// Quick start with BoxVector
memory = aiMemory( memory: "boxvector", config: {
    collection: "dev",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small"
} )

// Multi-tenant development testing
memory = aiMemory( memory: "boxvector",
    key: createUUID(),
    userId: "dev-user-123",
    conversationId: "test-chat",
    config: {
        collection: "dev_shared",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small"
    }
)

// Or use Hybrid for realistic testing
memory = aiMemory( memory: "hybrid", config: {
    recentLimit: 3,
    semanticLimit: 3,
    vectorProvider: "boxvector"
} )
```

### Production (Cloud)

```java
// Pinecone (managed) with multi-tenant isolation
memory = aiMemory( memory: "pinecone",
    key: createUUID(),
    userId: session.userId,
    conversationId: request.chatId,
    config: {
        collection: "prod_chat",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        apiKey: getSystemSetting( "PINECONE_API_KEY" ),
        environment: "us-west1-gcp",
        dimensions: 1536
    }
)

// Qdrant Cloud with multi-tenant isolation
memory = aiMemory( memory: "qdrant",
    key: createUUID(),
    userId: session.userId,
    conversationId: request.chatId,
    config: {
        collection: "prod_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "xyz.qdrant.io",
        port: 6333,
        apiKey: getSystemSetting( "QDRANT_API_KEY" ),
        https: true
    }
)
```

### Production (Self-Hosted)

```java
// PostgreSQL with pgvector and multi-tenant isolation
memory = aiMemory( memory: "postgres",
    key: createUUID(),
    userId: session.userId,
    conversationId: request.chatId,
    config: {
        collection: "ai_vectors",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        datasource: "mainDB",
        dimensions: 1536,
        autoCreate: true
    }
)

// Qdrant (Docker) with multi-tenant isolation
memory = aiMemory( memory: "qdrant",
    key: createUUID(),
    userId: session.userId,
    conversationId: request.chatId,
    config: {
        collection: "self_hosted_chat",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "qdrant.internal.network",
        port: 6333
    }
)
```

### Embedding Provider Options

```java
// OpenAI (fastest, most accurate)
{
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small"  // Or "text-embedding-3-large"
}

// Ollama (local, free)
{
    embeddingProvider: "ollama",
    embeddingModel: "nomic-embed-text"        // Or "mxbai-embed-large"
}

// Any supported provider
{
    embeddingProvider: "deepseek",
    embeddingModel: "deepseek-embedding"
}
```

### With Caching

```java
memory = aiMemory( memory: "chroma", config: {
    collection: "cached_chat",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    cache: true,                    // Enable embedding cache
    cacheName: "default",           // CacheBox provider
    cacheTimeout: 3600,             // 1 hour
    cacheLastAccessTimeout: 1800    // 30 minutes
} )
```

