---
description: >-
  Security considerations and performance optimization strategies for
  multi-tenant memory isolation.
icon: shield-halved
---

# Security & Performance

## Security Considerations

### 1. Always Validate User Identifiers

Never trust client-provided userId/conversationId without server-side validation:

```java
// ❌ WRONG: Direct use of untrusted input
memory = aiMemory( memory: "window",
    userId: url.userId,  // Client could manipulate this!
    config: { maxMessages: 10 }
)

// ✅ CORRECT: Server-side validation
function getUserMemory( required string requestedUserId ) {
    // Verify authenticated user matches requested user
    if ( session.user.id != arguments.requestedUserId ) {
        throw( type="SecurityViolation", message="Unauthorized access" );
    }

    return aiMemory( memory: "window",
        userId: session.user.id,  // Use authenticated session
        config: { maxMessages: 10 }
    );
}
```

### 2. Use Session-Based UserId

Always derive userId from authenticated session:

```java
function getAuthenticatedMemory( required string conversationId ) {
    // Ensure user is authenticated
    if ( !session.keyExists( "user" ) || !session.user.isAuthenticated ) {
        throw( type="Unauthorized", message="Login required" );
    }

    return aiMemory( memory: "session",
        key: "chat",
        userId: session.user.id,  // From authenticated session
        conversationId: arguments.conversationId,
        config: { maxMessages: 20 }
    );
}
```

### 3. Implement Authorization Checks

Verify user has permission to access specific conversations:

```java
function getConversationMemory(
    required string userId,
    required string conversationId
) {
    // Verify authenticated user matches userId
    if ( session.user.id != arguments.userId ) {
        throw( type="Unauthorized", message="Access denied" );
    }

    // Verify user owns this conversation
    var conversation = queryExecute(
        "SELECT user_id FROM conversations
         WHERE id = :conversationId AND user_id = :userId",
        {
            conversationId: arguments.conversationId,
            userId: arguments.userId
        }
    );

    if ( conversation.recordCount == 0 ) {
        throw( type="NotFound", message="Conversation not found" );
    }

    return aiMemory( memory: "jdbc",
        key: createUUID(),
        userId: arguments.userId,
        conversationId: arguments.conversationId,
        config: {
            datasource: "mainDB",
            table: "ai_conversations"
        }
    );
}
```

### 4. Sanitize Identifiers

Prevent injection attacks by sanitizing userId/conversationId:

```java
function sanitizeIdentifier( required string input ) {
    // Allow only alphanumeric, hyphens, underscores
    return reReplace( arguments.input, "[^a-zA-Z0-9\-_]", "", "ALL" );
}

function getSafeMemory( required string userId, required string conversationId ) {
    return aiMemory( memory: "file",
        key: createUUID(),
        userId: sanitizeIdentifier( arguments.userId ),
        conversationId: sanitizeIdentifier( arguments.conversationId ),
        config: {
            directoryPath: "/secure/memories",
            maxMessages: 50
        }
    );
}
```

### 5. Log Access for Auditing

Track who accesses which conversations:

```java
function getAuditedMemory(
    required string userId,
    required string conversationId
) {
    // Log access
    writeLog(
        type: "information",
        file: "memory-access",
        text: "User #arguments.userId# accessed conversation #arguments.conversationId# from IP #cgi.remote_addr#"
    );

    return aiMemory( memory: "jdbc",
        key: createUUID(),
        userId: arguments.userId,
        conversationId: arguments.conversationId,
        config: {
            datasource: "mainDB",
            table: "ai_conversations"
        }
    );
}
```

### 6. Implement Rate Limiting

Prevent abuse by limiting conversation access:

```java
class {
    property name="accessCounts" default="{}";

    function getRateLimitedMemory( required string userId ) {
        // Track access count
        if ( !variables.accessCounts.keyExists( arguments.userId ) ) {
            variables.accessCounts[ arguments.userId ] = {
                count: 0,
                resetAt: dateAdd( "h", 1, now() )
            };
        }

        var userAccess = variables.accessCounts[ arguments.userId ];

        // Reset if expired
        if ( now() > userAccess.resetAt ) {
            userAccess.count = 0;
            userAccess.resetAt = dateAdd( "h", 1, now() );
        }

        // Check limit
        if ( userAccess.count >= 100 ) {
            throw( type="RateLimitExceeded", message="Too many requests" );
        }

        userAccess.count++;

        return aiMemory( memory: "session",
            userId: arguments.userId,
            config: { maxMessages: 20 }
        );
    }
}
```

***


## Performance Optimization

### 1. Use Appropriate Memory Types

Choose memory types based on scale:

```java
// Small scale (< 100 users)
memory = aiMemory( memory: "session",
    userId: session.user.id,
    config: { maxMessages: 20 }
)

// Medium scale (100-10,000 users)
memory = aiMemory( memory: "cache",
    userId: session.user.id,
    config: {
        cacheName: "redis",
        maxMessages: 30
    }
)

// Large scale (> 10,000 users)
memory = aiMemory( memory: "jdbc",
    userId: session.user.id,
    config: {
        datasource: "mainDB",
        table: "ai_conversations",
        maxMessages: 100
    }
)
```

### 2. Index Database Columns

For JDBC and vector memory, ensure proper indexing:

```sql
-- PostgreSQL / MySQL
CREATE INDEX idx_user_id ON ai_conversations(user_id);
CREATE INDEX idx_conversation_id ON ai_conversations(conversation_id);
CREATE INDEX idx_composite ON ai_conversations(user_id, conversation_id);

-- For vector tables
CREATE INDEX idx_vector_user ON ai_vectors(user_id);
CREATE INDEX idx_vector_conversation ON ai_vectors(conversation_id);
CREATE INDEX idx_vector_composite ON ai_vectors(user_id, conversation_id);
```

### 3. Implement Caching

Cache memory instances to avoid repeated creation:

```java
class {
    property name="memoryCache" default="{}";

    function getOptimizedMemory( required string userId, required string conversationId ) {
        var cacheKey = "#arguments.userId#:#arguments.conversationId#";

        if ( !variables.memoryCache.keyExists( cacheKey ) ) {
            variables.memoryCache[ cacheKey ] = aiMemory( memory: "jdbc",
                key: createUUID(),
                userId: arguments.userId,
                conversationId: arguments.conversationId,
                config: {
                    datasource: "mainDB",
                    table: "ai_conversations",
                    maxMessages: 50
                }
            );
        }

        return variables.memoryCache[ cacheKey ];
    }
}
```

### 4. Cleanup Inactive Conversations

Periodically remove old conversations:

```java
function cleanupInactiveConversations( numeric daysInactive = 30 ) {
    var cutoffDate = dateAdd( "d", -arguments.daysInactive, now() );

    queryExecute(
        "DELETE FROM ai_conversations
         WHERE created_at < :cutoffDate",
        { cutoffDate: cutoffDate }
    );

    // Also cleanup vector memory if using database-backed provider
    queryExecute(
        "DELETE FROM ai_vectors
         WHERE created_at < :cutoffDate",
        { cutoffDate: cutoffDate }
    );
}
```

### 5. Use Connection Pooling

For database-backed memory, configure connection pooling:

```json
{
    "runtime": {
        "datasources": {
            "mainDB": {
                "driver": "mysql",
                "connectionString": "jdbc:mysql://localhost:3306/mydb",
                "username": "user",
                "password": "pass",
                "maxConnections": 50,
                "minConnections": 10,
                "connectionTimeout": 30000
            }
        }
    }
}
```

***

