---
description: >-
  Configuration and setup for managed cloud vector database providers:
  Pinecone, Qdrant, Weaviate, and Milvus.
icon: cloud
---

# Cloud Vector Providers

These providers are typically managed cloud services, offering hosted infrastructure with minimal operational overhead.

***

## PineconeVectorMemory

[Pinecone](https://www.pinecone.io/) managed cloud vector database.

**Features:**

* Fully managed, no ops
* Excellent performance
* Auto-scaling
* Built-in metadata filtering

**Setup:**

1. Sign up at [pinecone.io](https://www.pinecone.io/)
2. Create an index
3. Get API key

**Configuration:**

```java
memory = aiMemory( memory: "pinecone", config: {
    collection: "prod_conversations",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    apiKey: "your-pinecone-api-key",        // Or use PINECONE_API_KEY env var
    environment: "us-west1-gcp",            // Your Pinecone environment
    projectId: "your-project-id",           // Optional: project ID
    dimensions: 1536,
    metric: "cosine"
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "pinecone",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        apiKey: "your-pinecone-api-key",
        environment: "us-west1-gcp"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "pinecone",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        apiKey: "your-pinecone-api-key",
        environment: "us-west1-gcp"
    }
)

// Access identifiers
userId = memory.getUserId()
conversationId = memory.getConversationId()

// Export includes identifiers
exported = memory.export()
// { userId: "user123", conversationId: "chat456", ... }
```

**Best For:**

* Production cloud deployments
* Teams without ML ops expertise
* Rapid scaling requirements
* Global deployments

**Pricing:**

* Free tier: 1GB storage, 100K operations/month
* Paid: Scales with usage

***

## QdrantVectorMemory

[Qdrant](https://qdrant.tech/) high-performance vector search engine.

**Features:**

* Rust-based (excellent performance)
* Rich filtering capabilities
* Payload support
* Self-hosted or cloud

**Setup:**

```bash
# Docker
docker run -p 6333:6333 qdrant/qdrant

# Or Qdrant Cloud - sign up at qdrant.tech
```

**Configuration:**

```java
memory = aiMemory( memory: "qdrant", config: {
    collection: "chat_history",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    host: "localhost",              // Or Qdrant Cloud URL
    port: 6333,
    apiKey: "",                     // For Qdrant Cloud
    https: false,                   // Use true for cloud
    dimensions: 1536,
    metric: "cosine"
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "qdrant",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 6333
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "qdrant",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 6333
    }
)

// Access identifiers
userId = memory.getUserId()
conversationId = memory.getConversationId()

// Export includes identifiers
exported = memory.export()
// { userId: "user123", conversationId: "chat456", ... }
```

**Best For:**

* High-performance requirements
* Self-hosted production
* Complex filtering needs
* Large datasets (millions of vectors)

**Qdrant Cloud:**

* Free tier: 1GB cluster
* Excellent developer experience

***

## WeaviateVectorMemory

[Weaviate](https://weaviate.io/) GraphQL vector database with knowledge graph capabilities.

**Features:**

* GraphQL API
* Automatic vectorization (optional)
* Knowledge graph functionality
* Rich schema support

**Setup:**

```bash
# Docker
docker run -p 8080:8080 semitechnologies/weaviate:latest

# Or Weaviate Cloud
```

**Configuration:**

```java
memory = aiMemory( memory: "weaviate", config: {
    collection: "Conversations",    // Note: PascalCase for Weaviate classes
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    host: "localhost",              // Or WCS cluster URL
    port: 8080,
    scheme: "http",                 // Use "https" for WCS
    apiKey: "",                     // For Weaviate Cloud
    dimensions: 1536
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "weaviate",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "SharedCollection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 8080,
        scheme: "http"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "weaviate",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "AllConversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 8080,
        scheme: "http"
    }
)

// Access identifiers
userId = memory.getUserId()
conversationId = memory.getConversationId()

// Export includes identifiers
exported = memory.export()
// { userId: "user123", conversationId: "chat456", ... }
```

**Best For:**

* Complex entity relationships
* Knowledge graph requirements
* GraphQL preferences
* Multi-modal applications

***

## MilvusVectorMemory

[Milvus](https://milvus.io/) enterprise-grade distributed vector database.

**Features:**

* Massive scalability (billions of vectors)
* Distributed architecture
* GPU acceleration support
* Enterprise features

**Setup:**

```bash
# Docker Compose (Milvus Standalone)
wget https://github.com/milvus-io/milvus/releases/download/v2.3.0/milvus-standalone-docker-compose.yml -O docker-compose.yml
docker-compose up -d

# Or Zilliz Cloud (managed Milvus)
```

**Configuration:**

```java
memory = aiMemory( memory: "milvus", config: {
    collection: "enterprise_conversations",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    host: "localhost",
    port: 19530,
    user: "",                       // Optional authentication
    password: "",
    dimensions: 1536,
    metric: "IP",                   // Inner Product (or "L2", "COSINE")
    indexType: "IVF_FLAT"          // Index type for performance
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "milvus",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 19530,
        metric: "IP"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "milvus",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 19530,
        metric: "IP"
    }
)

// Access identifiers
userId = memory.getUserId()
conversationId = memory.getConversationId()

// Export includes identifiers
exported = memory.export()
// { userId: "user123", conversationId: "chat456", ... }
```

**Best For:**

* Enterprise deployments
* Massive datasets (> 10M vectors)
* High throughput requirements
* GPU-accelerated search

