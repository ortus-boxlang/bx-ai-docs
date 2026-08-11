---
description: >-
  Enterprise multi-tenant patterns, migrating existing memory to multi-tenant
  isolation, and troubleshooting common issues.
icon: building-shield
---

# Enterprise & Migration

## Enterprise Patterns

### Pattern 1: Multi-Organization SaaS

Isolate by organization → user → conversation:

```java
class {
    function getEnterpriseMemory(
        required string organizationId,
        required string userId,
        required string conversationId
    ) {
        // Validate organization membership
        if ( !userBelongsToOrganization( arguments.userId, arguments.organizationId ) ) {
            throw( type="Unauthorized", message="User not in organization" );
        }

        // Use composite userId for complete isolation
        var isolatedUserId = "#arguments.organizationId#:#arguments.userId#";

        return aiMemory( memory: "postgres",
            key: createUUID(),
            userId: isolatedUserId,
            conversationId: arguments.conversationId,
            config: {
                collection: "org_#arguments.organizationId#_vectors",
                datasource: "mainDB",
                embeddingProvider: "openai"
            }
        );
    }

    private boolean function userBelongsToOrganization(
        required string userId,
        required string organizationId
    ) {
        var result = queryExecute(
            "SELECT 1 FROM organization_users
             WHERE user_id = :userId AND organization_id = :organizationId",
            {
                userId: arguments.userId,
                organizationId: arguments.organizationId
            }
        );
        return result.recordCount > 0;
    }
}
```

### Pattern 2: Customer Support Ticketing

Map conversations to support tickets:

```java
class {
    function getTicketMemory( required string ticketId ) {
        // Get ticket details
        var ticket = getTicketById( arguments.ticketId );

        // Verify access
        if ( session.user.id != ticket.customerId &&
             !session.user.hasRole( "support" ) ) {
            throw( type="Unauthorized", message="Access denied" );
        }

        return aiMemory( memory: "hybrid",
            key: createUUID(),
            userId: ticket.customerId,
            conversationId: arguments.ticketId,
            config: {
                recentLimit: 10,
                semanticLimit: 5,
                vectorProvider: "pinecone",
                vectorConfig: {
                    collection: "support_history",
                    embeddingProvider: "openai"
                }
            }
        );
    }

    function createTicket( required string customerId, required string subject ) {
        var ticketId = createUUID();

        queryExecute(
            "INSERT INTO support_tickets (id, customer_id, subject, created_at)
             VALUES (:id, :customerId, :subject, :createdAt)",
            {
                id: ticketId,
                customerId: arguments.customerId,
                subject: arguments.subject,
                createdAt: now()
            }
        );

        return getTicketMemory( ticketId );
    }
}
```

### Pattern 3: Departmental Isolation

Separate conversations by department:

```java
function getDepartmentMemory(
    required string userId,
    required string department
) {
    // Verify user is in department
    if ( !userInDepartment( arguments.userId, arguments.department ) ) {
        throw( type="Unauthorized", message="Not authorized for this department" );
    }

    return aiMemory( memory: "chroma",
        key: createUUID(),
        userId: "#arguments.department#:#arguments.userId#",
        conversationId: "dept-chat",
        config: {
            collection: "dept_#arguments.department#_vectors",
            embeddingProvider: "openai"
        }
    );
}
```

***


## Migration Guide

### Migrating from Non-Multi-Tenant Memory

If you have existing non-multi-tenant memory implementations:

#### Step 1: Identify Current Usage

```java
// Old (single-tenant)
memory = aiMemory( memory: "window", config: { maxMessages: 10 } )
agent = aiAgent( name: "Assistant", memory: memory )
```

#### Step 2: Add UserId Parameter

```java
// New (multi-tenant)
memory = aiMemory( memory: "window",
    key: createUUID(),
    userId: session.user.id,  // Add user identifier
    config: { maxMessages: 10 }
)
agent = aiAgent( name: "Assistant", memory: memory )
```

#### Step 3: Update Existing Data

For database-backed memory (JDBC, Postgres, MySQL):

```sql
-- Add columns if missing
ALTER TABLE ai_conversations ADD COLUMN user_id VARCHAR(100);
ALTER TABLE ai_conversations ADD COLUMN conversation_id VARCHAR(100);

-- Migrate existing data (example: assign to default user)
UPDATE ai_conversations
SET user_id = 'legacy-user',
    conversation_id = 'default'
WHERE user_id IS NULL;

-- Add indexes
CREATE INDEX idx_user_id ON ai_conversations(user_id);
CREATE INDEX idx_conversation_id ON ai_conversations(conversation_id);
```

#### Step 4: Update Application Code

```java
// Before
function chat( message ) {
    var memory = aiMemory( memory: "window", config: { maxMessages: 10 } );
    var agent = aiAgent( name: "Bot", memory: memory );
    return agent.run( message );
}

// After
function chat( userId, conversationId, message ) {
    var memory = aiMemory( memory: "window",
        key: createUUID(),
        userId: arguments.userId,
        conversationId: arguments.conversationId,
        config: { maxMessages: 10 }
    );
    var agent = aiAgent( name: "Bot", memory: memory );
    return agent.run( message );
}
```

***


## Troubleshooting

### Issue: Users Seeing Other Users' Conversations

**Symptoms:**

* User A sees User B's conversation history
* Conversations mixing between users

**Solution:**

```java
// Ensure you're passing userId correctly
memory = aiMemory( memory: "window",
    key: createUUID(),
    userId: session.user.id,  // ✅ Use authenticated session
    config: { maxMessages: 10 }
)

// NOT this:
// userId: url.userId  ❌ Never trust client input
```

### Issue: Conversations Not Isolated Within User

**Symptoms:**

* User's different chat windows share history
* Multiple conversations bleeding together

**Solution:**

```java
// Add conversationId for isolation
memory = aiMemory( memory: "window",
    key: createUUID(),
    userId: session.user.id,
    conversationId: request.chatId,  // ✅ Add conversation identifier
    config: { maxMessages: 10 }
)
```

### Issue: Poor Performance with Many Users

**Symptoms:**

* Slow memory retrieval
* Database queries timing out

**Solutions:**

1. **Add database indexes:**

```sql
CREATE INDEX idx_composite ON ai_conversations(user_id, conversation_id);
```

2. **Use cache-based memory:**

```java
memory = aiMemory( memory: "cache",
    userId: session.user.id,
    config: {
        cacheName: "redis",
        maxMessages: 30
    }
)
```

3. **Implement memory pooling:**

```java
// Cache memory instances
if ( !application.memoryPool.keyExists( userId ) ) {
    application.memoryPool[ userId ] = aiMemory( memory: "window",
        userId: userId,
        config: { maxMessages: 10 }
    );
}
return application.memoryPool[ userId ];
```

### Issue: Memory Leakage Between Tenants

**Symptoms:**

* Data from other users occasionally appears
* Intermittent cross-contamination

**Solution:**

```java
// Ensure you're creating NEW memory instances, not reusing
function getUserMemory( userId ) {
    // ❌ WRONG: Reusing same instance
    // return variables.sharedMemory;

    // ✅ CORRECT: New instance per user
    return aiMemory( memory: "session",
        key: createUUID(),  // Unique key per instance
        userId: arguments.userId,
        config: { maxMessages: 20 }
    );
}
```

### Issue: Export/Import Losing Tenant Information

**Symptoms:**

* Imported conversations lose userId/conversationId
* Restored memory shows wrong owner

**Solution:**

```java
// Export preserves identifiers
exported = memory.export();
// {
//     userId: "user123",
//     conversationId: "chat456",
//     messages: [...]
// }

// Import restores identifiers
newMemory = aiMemory( memory: "window", config: {
    maxMessages: 10
});
newMemory.import( exported );  // Preserves userId/conversationId

// Verify restoration
println( newMemory.getUserId() );  // "user123"
println( newMemory.getConversationId() );  // "chat456"
```

***

