---
description: >-
  Isolating user and tenant data completely across memory, namespaces, and row-level database security.
icon: users
---

# Multi-Tenant Security

### Complete Isolation

**Ensure users can only access their own data**:

```javascript
class {
    function getUserMemory( required string userId ) {
        // CRITICAL: Always filter by authenticated user ID
        // Never trust client-provided user IDs

        var authenticatedUserId = session.user.id

        // Verify authorization
        if ( arguments.userId != authenticatedUserId ) {
            writeLog(
                "Unauthorized memory access attempt: user #authenticatedUserId# tried to access #arguments.userId#",
                "security"
            )
            throw "Unauthorized access"
        }

        // Return memory scoped to user
        return aiMemory( memory: "cache", config: {
            namespace: "user_#authenticatedUserId#"
        } )
    }

    function chatWithMemory( required string prompt, required string userId ) {
        var memory = getUserMemory( arguments.userId )

        return aiChat(
            arguments.prompt,
            { memory: memory }
        )
    }
}
```

### Namespace Isolation

```javascript
class {
    function getTenantNamespace( required string tenantId ) {
        // Hash tenant ID for additional security
        return "tenant_" & hash( arguments.tenantId, "SHA-256" )
    }

    function getTenantMemory( required string tenantId ) {
        var namespace = getTenantNamespace( arguments.tenantId )

        return aiMemory( memory: "jdbc", config: {
            namespace: namespace,
            tableName: "ai_memory_#namespace#"
        } )
    }

    function getTenantVectorMemory( required string tenantId ) {
        var namespace = getTenantNamespace( arguments.tenantId )

        return aiVectorMemory( "chroma", {
            collectionName: namespace
        } )
    }
}
```

### Row-Level Security

**For database-backed memory**:

```sql
-- PostgreSQL Row-Level Security (RLS)
CREATE TABLE ai_memory (
    id SERIAL PRIMARY KEY,
    tenant_id VARCHAR(255) NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    content JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE ai_memory ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own data
CREATE POLICY tenant_isolation ON ai_memory
    USING (tenant_id = current_setting('app.tenant_id'));

-- Set tenant context in application
SET app.tenant_id = 'tenant-123';
```

```javascript
// BoxLang application code
function setTenantContext( required string tenantId ) {
    queryExecute(
        "SET app.tenant_id = :tenantId",
        { tenantId: arguments.tenantId }
    )
}

function getUserMemory( required string userId ) {
    // Set tenant context first
    setTenantContext( session.tenantId )

    // Query automatically filtered by RLS
    var messages = queryExecute(
        "SELECT content FROM ai_memory WHERE user_id = :userId",
        { userId: arguments.userId }
    )

    return messages
}
```
