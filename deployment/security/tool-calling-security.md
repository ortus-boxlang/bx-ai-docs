---
description: >-
  Validating tool and function-calling parameters and sandboxing tool execution against prompt-injection driven misuse.
icon: wrench
---

# Tool & Function Calling Security

{% hint style="info" %}
`GuardrailMiddleware` blocks dangerous **tool calls** by name, or validates their arguments against regex patterns, before any tool runs — often simpler than the parameter-validation code below. See [Middleware](../../main-components/middleware.md#guardrailmiddleware).
{% endhint %}

### The Tool Calling Risk

**AI agents can autonomously invoke tools based on user requests**. If inputs aren't validated, attackers can:

- **Trigger unintended tool calls**: `"Search my entire database"` → database lookup tool
- **Pass malicious parameters**: `"Look up user with id: 1; DROP TABLE users; --"`
- **Exploit tool side effects**: Delete files, transfer funds, send emails
- **Combine tools maliciously**: Web search → database lookup → email tool chain

### Parameter Validation Before Tool Execution

```javascript
class {
    function safeWebSearchTool( required string query ) {
        // ✅ Validate before passing to web search
        if ( !validateSearchQuery( arguments.query ) ) {
            throw "Invalid search query"
        }

        // Limit result count
        var maxResults = 10

        var results = aiWebSearch(
            arguments.query,
            { maxResults: maxResults }
        )

        // Validate results before returning to AI
        return validateSearchResults( results )
    }

    function validateSearchQuery( required string query ) {
        // Check length
        if ( len( arguments.query ) > 1000 ) {
            return false
        }

        // Check for SQL injection patterns
        if ( arguments.query.findNoCase( "DROP" ) > 0 ||
             arguments.query.findNoCase( "DELETE" ) > 0 ) {
            return false
        }

        // Check for command injection
        if ( arguments.query.find( "&&" ) > 0 ||
             arguments.query.find( "|" ) > 0 ) {
            return false
        }

        return true
    }

    function validateSearchResults( required array results ) {
        // Filter out suspicious domains
        var blockedDomains = [ "malware-site.com", "phishing.net" ]

        return results.filter( r => {
            var domain = extractDomain( r.url )
            return !blockedDomains.contains( domain )
        } )
    }
}
```

### Tool Invocation Sandboxing

```javascript
class {
    function createSandboxedTool( required function toolFn, required struct schema ) {
        return aiTool(
            schema.name,
            schema.description,
            ( args ) => {
                // Validate input against schema
                validateToolInput( args, schema )

                // Execute in isolation with error handling
                try {
                    var result = toolFn( args )

                    // Validate output
                    if ( len( result ) > 10000 ) {
                        return "[Result truncated - output too large]"
                    }

                    return result

                } catch ( any e ) {
                    // Don't leak error details to AI
                    logError( e, args )
                    return "[Tool execution failed]"
                }
            }
        )
    }

    function validateToolInput( required struct args, required struct schema ) {
        for ( param in schema.parameters ?: [] ) {
            if ( param.required && !structKeyExists( arguments.args, param.name ) ) {
                throw "Missing required parameter: #param.name#"
            }

            if ( structKeyExists( arguments.args, param.name ) ) {
                var value = arguments.args[ param.name ]
                var type = param.type ?: "string"

                // Type validation
                if ( type == "string" && !isSimpleValue( value ) ) {
                    throw "Parameter #param.name# must be a string"
                }

                if ( type == "number" && !isNumeric( value ) ) {
                    throw "Parameter #param.name# must be numeric"
                }

                // Length limits
                if ( type == "string" && len( value ) > (param.maxLength ?: 1000) ) {
                    throw "Parameter #param.name# exceeds maximum length"
                }
            }
        }
    }
}
```

### Tool Audit & Rate Limiting

```javascript
class {
    function logToolExecution(
        required string toolName,
        required string userId,
        required struct parameters,
        any result,
        numeric durationMs = 0
    ) {
        queryExecute(
            "INSERT INTO tool_audit_log (tool_name, user_id, parameters, result, duration_ms, created_at)
             VALUES (:toolName, :userId, :parameters, :result, :durationMs, :createdAt)",
            {
                toolName: arguments.toolName,
                userId: arguments.userId,
                parameters: jsonSerialize( arguments.parameters ),
                result: jsonSerialize( arguments.result ?: {} ),
                durationMs: arguments.durationMs,
                createdAt: now()
            }
        )

        // Alert on suspicious patterns
        if ( arguments.toolName == "aiWebSearch" && arguments.durationMs > 5000 ) {
            writeLog( "Slow web search detected: #arguments.durationMs#ms", "warning" )
        }
    }

    function checkToolRateLimit( required string userId, required string toolName ) {
        var query = queryExecute(
            "SELECT COUNT(*) as cnt FROM tool_audit_log
             WHERE user_id = :userId
             AND tool_name = :toolName
             AND created_at > DATE_SUB(NOW(), INTERVAL 1 MINUTE)",
            {
                userId: arguments.userId,
                toolName: arguments.toolName
            }
        )

        var callsPerMinute = query.cnt
        var limit = 10  // 10 calls per minute per tool

        if ( callsPerMinute >= limit ) {
            throw "Rate limit exceeded for tool: #arguments.toolName#"
        }
    }
}
```
