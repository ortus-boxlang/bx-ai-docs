---
description: >-
  Comprehensive audit logging for AI interactions, plus a query API for reviewing activity.
icon: clipboard-list
---

# Audit Logging

### Comprehensive Logging

**Log all AI interactions for security and compliance**:

```javascript
class {
    function logAIInteraction(
        required string action,
        required string userId,
        required string prompt,
        string response = "",
        struct metadata = {}
    ) {
        var logEntry = {
            timestamp: now(),
            action: arguments.action,
            userId: arguments.userId,
            promptHash: hash( arguments.prompt, "SHA-256" ),
            promptLength: len( arguments.prompt ),
            responseLength: len( arguments.response ),
            provider: arguments.metadata.provider ?: "",
            model: arguments.metadata.model ?: "",
            tokens: arguments.metadata.tokens ?: 0,
            cost: arguments.metadata.cost ?: 0,
            success: arguments.metadata.success ?: true,
            errorType: arguments.metadata.errorType ?: "",
            ipAddress: cgi.remote_addr,
            userAgent: cgi.http_user_agent,
            requestId: request.requestId ?: createUUID()
        }

        // Write to audit log
        writeLog(
            text: jsonSerialize( logEntry ),
            type: "audit",
            file: "ai-audit"
        )

        // Also store in database for querying
        queryExecute(
            "INSERT INTO ai_audit_log (data, created_at) VALUES (:data, :createdAt)",
            {
                data: jsonSerialize( logEntry ),
                createdAt: now()
            }
        )
    }

    function aiChatWithAudit(
        required string prompt,
        struct params = {}
    ) {
        var userId = session.user.id
        var startTime = getTickCount()

        try {
            // Log request
            logAIInteraction(
                action: "ai_chat_request",
                userId: userId,
                prompt: arguments.prompt,
                metadata: {
                    provider: arguments.params.provider ?: "openai",
                    model: arguments.params.model ?: "gpt-4"
                }
            )

            // Call AI
            var response = aiChat( arguments.prompt, arguments.params )

            // Log response
            logAIInteraction(
                action: "ai_chat_response",
                userId: userId,
                prompt: arguments.prompt,
                response: response,
                metadata: {
                    provider: arguments.params.provider ?: "openai",
                    model: arguments.params.model ?: "gpt-4",
                    duration: getTickCount() - startTime,
                    success: true
                }
            )

            return response

        } catch ( any e ) {
            // Log error
            logAIInteraction(
                action: "ai_chat_error",
                userId: userId,
                prompt: arguments.prompt,
                metadata: {
                    provider: arguments.params.provider ?: "openai",
                    errorType: e.type,
                    errorMessage: e.message,
                    success: false
                }
            )

            throw e
        }
    }
}
```

### Audit Query API

```javascript
class {
    function getAuditLogs(
        string userId = "",
        string action = "",
        date startDate,
        date endDate,
        numeric limit = 100
    ) {
        var sql = "SELECT * FROM ai_audit_log WHERE 1=1"
        var params = {}

        if ( len( arguments.userId ) > 0 ) {
            sql &= " AND data->>'userId' = :userId"
            params.userId = arguments.userId
        }

        if ( len( arguments.action ) > 0 ) {
            sql &= " AND data->>'action' = :action"
            params.action = arguments.action
        }

        if ( !isNull( arguments.startDate ) ) {
            sql &= " AND created_at >= :startDate"
            params.startDate = arguments.startDate
        }

        if ( !isNull( arguments.endDate ) ) {
            sql &= " AND created_at <= :endDate"
            params.endDate = arguments.endDate
        }

        sql &= " ORDER BY created_at DESC LIMIT :limit"
        params.limit = arguments.limit

        return queryExecute( sql, params )
    }

    function getUserActivity( required string userId, numeric days = 30 ) {
        return getAuditLogs(
            userId: arguments.userId,
            startDate: dateAdd( "d", -arguments.days, now() )
        )
    }

    function getFailedRequests( numeric days = 7 ) {
        var sql = "SELECT * FROM ai_audit_log
                   WHERE data->>'success' = 'false'
                   AND created_at >= :startDate
                   ORDER BY created_at DESC"

        return queryExecute( sql, {
            startDate: dateAdd( "d", -arguments.days, now() )
        } )
    }
}
```
