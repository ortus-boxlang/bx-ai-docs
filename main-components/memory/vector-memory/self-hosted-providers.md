---
description: >-
  Configuration and setup for self-hosted vector memory providers: BoxVector,
  Chroma, Postgres, MySQL, OpenSearch, and TypeSense.
icon: server
---

# Self-Hosted Vector Providers

These providers are typically self-managed and run on your own infrastructure (local, on-prem, or self-hosted in the cloud).

***

## BoxVectorMemory

In-memory vector storage perfect for development and testing.

**Features:**

* No external dependencies
* Instant setup
* Full feature support
* Cosine similarity search

**Configuration:**

```java
memory = aiMemory( memory: "boxvector", config: {
    collection: "dev_chat",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    cache: true,               // Cache embeddings
    cacheName: "default"
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "boxvector",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "boxvector",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small"
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

* Local development
* Testing
* Small datasets (< 10,000 messages)
* Proof of concepts

**Limitations:**

* Data lost on restart
* Limited to single instance
* Memory usage grows with dataset

***

## ChromaVectorMemory

[ChromaDB](https://www.trychroma.com/) integration for local vector storage.

**Features:**

* Local persistence
* Python ecosystem integration
* Easy Docker deployment
* Metadata filtering

**Setup:**

```bash
# Docker
docker run -p 8000:8000 chromadb/chroma

# Or Python
pip install chromadb
chroma run --host 0.0.0.0 --port 8000
```

**Configuration:**

```java
memory = aiMemory( memory: "chroma", config: {
    collection: "customer_support",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    host: "localhost",         // ChromaDB host
    port: 8000,                // ChromaDB port
    protocol: "http",
    tenant: "default_tenant",
    database: "default_database",
    timeout: 30
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "chroma",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 8000
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "chroma",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 8000
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

* Python-based infrastructure
* Local development with persistence
* Medium datasets (< 1M vectors)

***

## PostgresVectorMemory

PostgreSQL with [pgvector](https://github.com/pgvector/pgvector) extension.

**Features:**

* Use existing Postgres infrastructure
* ACID compliance
* Familiar SQL queries
* Mature ecosystem

**Setup:**

```sql
-- Enable pgvector extension
CREATE EXTENSION vector;
```

**Configuration:**

```java
memory = aiMemory( memory: "postgres", config: {
    collection: "ai_memory",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    datasource: "myPostgresDS",     // JDBC datasource
    tableName: "vector_memory",
    dimensions: 1536,                // Embedding dimensions
    metric: "cosine",                // Distance metric
    autoCreate: true                 // Auto-create table
} )
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "postgres",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        datasource: "myPostgresDS",
        tableName: "vector_memory"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "postgres",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        datasource: "myPostgresDS"
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

* Existing PostgreSQL deployments
* Applications requiring SQL access
* Strong consistency requirements
* Medium-large datasets

***

## MysqlVectorMemory

MySQL 9+ with native [VECTOR](https://dev.mysql.com/doc/refman/9.0/en/vector-functions.html) data type support.

**Features:**

* Native vector storage (MySQL 9+)
* Use existing MySQL infrastructure
* ACID compliance
* Familiar SQL ecosystem
* Application-layer distance calculations (MySQL Community Edition compatible)

**Requirements:**

* MySQL 9.0 or later (Community or Enterprise Edition)
* Configured BoxLang datasource
* VECTOR data type support

**Setup:**

MySQL 9 Community Edition includes native VECTOR data type support. No extensions needed - tables are auto-created:

```sql
-- Tables are created automatically, but here's the structure:
CREATE TABLE bx_ai_vectors (
    id VARCHAR(255) PRIMARY KEY,
    text LONGTEXT NOT NULL,
    embedding VECTOR(1536) NOT NULL,  -- Native VECTOR type
    metadata JSON,
    collection VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_collection (collection)
);
```

**Configuration:**

```java
memory = aiMemory( memory: "mysql", config: {
    collection: "ai_memory",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    datasource: "myMysqlDS",         // JDBC datasource name
    table: "bx_ai_vectors",          // Optional: default is "bx_ai_vectors"
    dimensions: 1536,                // Embedding dimensions
    distanceFunction: "COSINE",      // L2, COSINE, or DOT
    autoCreate: true                 // Auto-create table (default: true)
} )
```

**BoxLang Datasource Setup:**

```json
// boxlang.json
{
    "runtime": {
        "datasources": {
            "myMysqlDS": {
                "driver": "mysql",
                "connectionString": "jdbc:mysql://localhost:3306/mydb",
                "username": "user",
                "password": "pass"
            }
        }
    }
}
```

**Distance Functions:**

* **COSINE**: Cosine distance (1 - cosine similarity), best for semantic search
* **L2**: Euclidean distance (L2 norm), good for spatial data
* **DOT**: Dot product similarity, efficient for normalized vectors

**Usage Example:**

```java
// Create MySQL vector memory
memory = aiMemory( memory: "mysql", config: {
    collection: "customer_support",
    datasource: "myMysqlDS",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    distanceFunction: "COSINE"
} )

// Use with agent
agent = aiAgent(
    name: "Support Bot",
    memory: memory
)

// Conversations are stored with vector embeddings
agent.run( "I need help with billing" )
agent.run( "What are the payment options?" )

// Semantically similar past conversations are automatically retrieved
agent.run( "Tell me about invoices" )  // Finds billing-related history
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "mysql",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        datasource: "myMysqlDS",
        distanceFunction: "COSINE"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "mysql",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        datasource: "myMysqlDS"
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

* Existing MySQL 9+ deployments
* Organizations standardized on MySQL
* Applications requiring SQL access
* ACID compliance requirements
* Medium-large datasets (millions of vectors)

**Performance Notes:**

* Distance calculations performed in application layer (MySQL Community Edition compatible)
* MySQL HeatWave (Oracle Cloud) provides native DISTANCE() function for optimal performance
* Suitable for production use with proper indexing
* Table is automatically created with collection-based indexing

**MySQL Community vs HeatWave:**

* **Community Edition** (Free): VECTOR data type, app-layer distance calculations
* **HeatWave** (Oracle Cloud): Native DISTANCE() function, VECTOR INDEX, GPU acceleration

***

## OpenSearchVectorMemory

[OpenSearch](https://opensearch.org/) distributed search and analytics engine with k-NN vector search capabilities.

**Features:**

* AWS Elasticsearch-compatible service
* k-NN vector search with HNSW algorithm
* Enterprise-grade security
* Multi-tenant isolation
* Advanced filtering and aggregations
* AWS integration (IAM, CloudWatch)

**Requirements:**

* OpenSearch 1.x+ or AWS OpenSearch Service
* HTTP/HTTPS access to cluster
* API credentials or AWS IAM authentication

**Setup:**

```bash
# Docker (quickest way)
docker run -p 9200:9200 -p 9600:9600 \
  -e "discovery.type=single-node" \
  -e "plugins.security.disabled=true" \
  opensearchproject/opensearch:latest

# Docker Compose
services:
  opensearch:
    image: opensearchproject/opensearch:latest
    environment:
      - discovery.type=single-node
      - plugins.security.disabled=true
    ports:
      - "9200:9200"
      - "9600:9600"

# Or use AWS OpenSearch Service (managed)
# Create domain in AWS Console or via Terraform/CloudFormation
```

**Configuration:**

```javascript
// Basic OpenSearch configuration
memory = aiMemory( memory: "opensearch", config: {
    collection: "ai_conversations",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    host: "localhost",               // OpenSearch host
    port: 9200,                      // Default OpenSearch port
    protocol: "http",                // Use "https" for AWS OpenSearch
    dimensions: 1536,                // Must match embedding model
    engine: "nmslib",                // Options: "nmslib", "faiss", "lucene"
    spaceType: "cosinesimil",        // Options: "l2", "cosinesimil", "innerproduct"
    m: 16,                           // HNSW algorithm parameter (higher = better accuracy)
    efConstruction: 512,             // HNSW construction parameter
    efSearch: 512                    // Search-time HNSW parameter
} )
```

**AWS OpenSearch Configuration:**

```javascript
// AWS OpenSearch Service
memory = aiMemory( memory: "opensearch", config: {
    collection: "production_memory",
    embeddingProvider: "bedrock",
    embeddingModel: "amazon.titan-embed-text-v1",
    host: "search-my-domain.us-west-2.es.amazonaws.com",
    port: 443,
    protocol: "https",
    username: "admin",               // Master user
    password: "Complex-Password123!", // From AWS Console
    region: "us-west-2",             // AWS region
    dimensions: 1536,
    engine: "faiss",                 // FAISS for AWS OpenSearch
    spaceType: "cosinesimil"
} )
```

**AWS IAM Authentication:**

```javascript
// Using AWS IAM credentials (no username/password)
memory = aiMemory( memory: "opensearch", config: {
    collection: "enterprise_memory",
    embeddingProvider: "bedrock",
    embeddingModel: "amazon.titan-embed-text-v1",
    host: "vpc-private-domain.us-west-2.es.amazonaws.com",
    port: 443,
    protocol: "https",
    region: "us-west-2",
    useIAM: true,                    // Enable IAM authentication
    accessKeyId: "AKIAIOSFODNN7EXAMPLE",      // AWS credentials
    secretAccessKey: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
    dimensions: 1536
} )
```

**Usage Example:**

```javascript
// Create OpenSearch vector memory
memory = aiMemory( memory: "opensearch", config: {
    collection: "customer_support",
    host: "localhost",
    port: 9200,
    protocol: "http",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    engine: "nmslib",
    spaceType: "cosinesimil"
} )

// Use with agent
agent = aiAgent(
    name: "Support Bot",
    memory: memory
)

// Semantic search with OpenSearch
agent.run( "How do I reset my password?" )
agent.run( "What are the payment options?" )
```

**Multi-Tenant Configuration:**

```javascript
// Per-user isolation
memory = aiMemory( memory: "opensearch",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 9200,
        protocol: "http",
        engine: "nmslib"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "opensearch",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "bedrock",
        embeddingModel: "amazon.titan-embed-text-v1",
        host: "search-domain.us-west-2.es.amazonaws.com",
        port: 443,
        protocol: "https",
        region: "us-west-2",
        username: "admin",
        password: "password"
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

* AWS infrastructure integration
* Enterprise search applications
* Large-scale vector search (billions of vectors)
* Applications requiring advanced filtering
* Organizations already using Elasticsearch/OpenSearch
* Compliance-heavy environments (HIPAA, SOC2)

**OpenSearch Advantages:**

* **AWS Integration**: Native IAM, CloudWatch, VPC support
* **Enterprise Features**: RBAC, audit logging, encryption at rest
* **Scalability**: Horizontal scaling, cluster management
* **Flexibility**: Multiple k-NN algorithms (HNSW, FAISS, Lucene)
* **Cost-Effective**: AWS reserved instances, spot instances

**Pricing:**

* **Self-Hosted**: Free (open source)
* **AWS OpenSearch Service**:
  * On-Demand: Per-hour instance pricing
  * Reserved Instances: Up to 72% savings
  * Storage: $0.135/GB-month (standard)

**When to Choose OpenSearch:**

* Already using AWS infrastructure
* Need enterprise-grade security and compliance
* Require advanced search capabilities (filtering, aggregations)
* Building large-scale applications (> 1B vectors)
* Need tight AWS service integration (Lambda, S3, etc.)

**Performance Notes:**

* HNSW algorithm provides excellent recall/performance balance
* FAISS engine for maximum performance on AWS
* Lucene engine for exact search (100% recall)
* Index configuration (m, efConstruction, efSearch) impacts performance vs accuracy

***

## TypesenseVectorMemory

[TypeSense](https://typesense.org/) is a fast, typo-tolerant search engine optimized for instant search experiences and vector similarity search.

**Features:**

* Lightning-fast search with typo tolerance
* Native vector search support
* Easy Docker deployment
* RESTful API
* Built-in relevance tuning
* Excellent for autocomplete and instant search

**Requirements:**

* TypeSense Server 0.23.0+ (vector search support)
* HTTP/HTTPS access to TypeSense instance
* API key for authentication

**Setup:**

```bash
# Docker (quickest way)
export TYPESENSE_API_KEY=xyz
docker run -p 8108:8108 \
  -v $(pwd)/typesense-data:/data \
  typesense/typesense:29.0 \
  --data-dir /data \
  --api-key=$TYPESENSE_API_KEY \
  --enable-cors

# Docker Compose
services:
  typesense:
    image: typesense/typesense:29.0
    restart: on-failure
    ports:
      - "8108:8108"
    volumes:
      - ./typesense-data:/data
    command: '--data-dir /data --api-key=xyz --enable-cors'

# Or use TypeSense Cloud (managed service)
# Sign up at https://cloud.typesense.org/
```

**Configuration:**

```java
memory = aiMemory( memory: "typesense", config: {
    collection: "ai_conversations",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    host: "localhost",               // TypeSense host
    port: 8108,                      // Default TypeSense port
    protocol: "http",                // Use "https" for TypeSense Cloud
    apiKey: "xyz",                   // Or use TYPESENSE_API_KEY env var
    dimensions: 1536,                // Must match embedding model
    timeout: 30                      // Connection timeout in seconds
} )
```

**TypeSense Cloud Configuration:**

```java
// For TypeSense Cloud (managed service)
memory = aiMemory( memory: "typesense", config: {
    collection: "production_memory",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small",
    host: "xxx.a1.typesense.net",    // From TypeSense Cloud dashboard
    port: 443,
    protocol: "https",
    apiKey: "your-cloud-api-key",    // From TypeSense Cloud dashboard
    dimensions: 1536
} )
```

**Usage Example:**

```java
// Create TypeSense vector memory
memory = aiMemory( memory: "typesense", config: {
    collection: "customer_support",
    host: "localhost",
    port: 8108,
    protocol: "http",
    apiKey: "xyz",
    embeddingProvider: "openai",
    embeddingModel: "text-embedding-3-small"
} )

// Use with agent
agent = aiAgent(
    name: "Support Bot",
    memory: memory
)

// Fast, typo-tolerant semantic search
agent.run( "How do I reset my pasword?" )  // Finds "password" results despite typo
agent.run( "What are the paiment options?" )  // Finds "payment" results
```

**Multi-Tenant Configuration:**

```java
// Per-user isolation
memory = aiMemory( memory: "typesense",
    key: createUUID(),
    userId: "user123",
    config: {
        collection: "shared_collection",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 8108,
        protocol: "http",
        apiKey: "xyz"
    }
)

// Per-conversation isolation
memory = aiMemory( memory: "typesense",
    key: createUUID(),
    userId: "user123",
    conversationId: "chat456",
    config: {
        collection: "all_conversations",
        embeddingProvider: "openai",
        embeddingModel: "text-embedding-3-small",
        host: "localhost",
        port: 8108,
        protocol: "http",
        apiKey: "xyz"
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

* Applications requiring fast, low-latency search
* Autocomplete and instant search features
* Typo-tolerant semantic search
* E-commerce product search
* Documentation search
* Customer support systems
* Small to medium datasets (< 10M vectors)

**TypeSense Advantages:**

* **Speed**: Sub-50ms search latency
* **Typo Tolerance**: Built-in fuzzy search
* **Simple Setup**: Single binary, easy Docker deployment
* **RESTful API**: Simple HTTP API, easy integration
* **Relevance Tuning**: Fine-grained control over ranking

**Pricing:**

* **Self-Hosted**: Free (open source)
* **TypeSense Cloud**:
  * Free tier: Development clusters
  * Paid: Production clusters from $0.03/hour

**When to Choose TypeSense:**

* Need instant search with typo tolerance
* Want simple deployment and management
* Require low-latency semantic search
* Building search-heavy applications
* Need both keyword and vector search

**Performance Notes:**

* Optimized for low-latency queries (< 50ms)
* In-memory index for fast access
* Horizontal scaling support
* Efficient resource usage

