---
description: >-
  Practical interceptor patterns for logging, cost tracking, caching, content
  filtering, fallback, A/B testing, and safety guardrails.
icon: lightbulb
---

# Common Use Cases & Examples

## Common Use Cases

### Request Logging and Monitoring

```java
function onAIChatRequest( event, interceptData ) {
    var logData = {
        timestamp: now(),
        provider: interceptData.provider.getProviderName(),
        messages: interceptData.aiRequest.getMessages(),
        params: interceptData.aiRequest.getParams(),
        user: getAuthenticatedUser()
    };

    // Log to database
    queryExecute(
        "INSERT INTO ai_request_log (data, created_at) VALUES (:data, :timestamp)",
        { data: serializeJSON( logData ), timestamp: now() }
    );
}
```

### Cost Tracking and Budgeting

```java
class {

    function onAIChatResponse( event, interceptData ) {
        if ( !interceptData.response.keyExists( "usage" ) ) return;

        var usage = interceptData.response.usage;
        var provider = interceptData.provider.getProviderName();

        // Calculate cost based on provider pricing
        var cost = calculateCost( provider, usage );

        // Track against user/org budget
        var user = getAuthenticatedUser();
        var currentUsage = getUserUsage( user );

        updateUserUsage( user, cost );

        // Alert if approaching limit
        if ( currentUsage + cost >= getUserBudget( user ) * 0.9 ) {
            sendBudgetAlert( user );
        }

        // Block if over budget
        if ( currentUsage + cost > getUserBudget( user ) ) {
            throw( "Budget exceeded for user: #user#" );
        }
    }

    private function calculateCost( provider, usage ) {
        // Pricing per 1K tokens (example rates)
        var pricing = {
            openai: {
                prompt: 0.03,
                completion: 0.06
            },
            claude: {
                prompt: 0.015,
                completion: 0.075
            }
        };

        var rates = pricing[ provider ] ?: { prompt: 0.01, completion: 0.01 };

        return (
            ( usage.prompt_tokens / 1000 ) * rates.prompt +
            ( usage.completion_tokens / 1000 ) * rates.completion
        );
    }
}
```

### Response Caching

```java
class {

    function onAIChatRequest( event, interceptData ) {
        var request = interceptData.aiRequest;
        var cacheKey = generateCacheKey( request );

        // Check cache
        var cached = cacheGet( cacheKey );
        if ( !isNull( cached ) ) {
            // Return cached response and skip actual API call
            interceptData.useCached = true;
            interceptData.cachedResponse = cached;

            writeLog(
                text: "Using cached AI response",
                type: "info"
            );
        }
    }

    function onAIChatResponse( event, interceptData ) {
        // Don't cache if we used cached response
        if ( interceptData.keyExists( "useCached" ) ) return;

        var request = interceptData.aiRequest;
        var response = interceptData.response;
        var cacheKey = generateCacheKey( request );

        // Cache for 1 hour
        cachePut(
            cacheKey,
            response,
            createTimeSpan( 0, 1, 0, 0 )
        );
    }

    private function generateCacheKey( request ) {
        var key = {
            messages: request.getMessages(),
            params: request.getParams()
        };
        return hash( serializeJSON( key ) );
    }
}
```

### Content Filtering and Moderation

```java
function onAIChatRequest( event, interceptData ) {
    var request = interceptData.aiRequest;
    var messages = request.getMessages();

    // Check for prohibited content
    for ( var msg in messages ) {
        if ( containsProhibitedContent( msg.content ) ) {
            throw(
                type: "ContentViolation",
                message: "Request contains prohibited content"
            );
        }
    }
}

function onAIChatResponse( event, interceptData ) {
    var response = interceptData.response;

    // Filter response content
    if ( response.keyExists( "choices" ) ) {
        for ( var choice in response.choices ) {
            if ( choice.keyExists( "message" ) ) {
                choice.message.content = filterContent(
                    choice.message.content
                );
            }
        }
    }
}

private function containsProhibitedContent( text ) {
    var prohibitedPatterns = [
        "pattern1",
        "pattern2"
    ];

    for ( var pattern in prohibitedPatterns ) {
        if ( findNoCase( pattern, text ) ) {
            return true;
        }
    }

    return false;
}

private function filterContent( text ) {
    // Replace sensitive information
    text = reReplace( text, "\b\d{3}-\d{2}-\d{4}\b", "[SSN]", "all" );
    text = reReplace( text, "\b\d{16}\b", "[CREDIT_CARD]", "all" );
    return text;
}
```

### Multi-Provider Fallback

```java
class {

    property name="failedProviders" default={};

    function onAIChatRequest( event, interceptData ) {
        var provider = interceptData.provider.getProviderName();

        // Check if provider is in cooldown
        if ( failedProviders.keyExists( provider ) ) {
            var cooldownEnd = failedProviders[ provider ];
            if ( now() < cooldownEnd ) {
                // Switch to backup provider
                var backup = getBackupProvider( provider );
                writeLog(
                    text: "Switching from #provider# to #backup# (cooldown)",
                    type: "warning"
                );
                // Recreate request with backup provider
                throw(
                    type: "ProviderCooldown",
                    message: "Provider in cooldown, use backup"
                );
            } else {
                // Cooldown expired, remove from list
                structDelete( failedProviders, provider );
            }
        }
    }

    function onAIChatResponse( event, interceptData ) {
        var response = interceptData.response;
        var provider = interceptData.provider.getProviderName();

        // Check for rate limiting
        if ( response.keyExists( "error" ) &&
             response.error.type == "rate_limit_exceeded" ) {

            // Put provider in 5-minute cooldown
            failedProviders[ provider ] = dateAdd( "n", 5, now() );

            writeLog(
                text: "Provider #provider# rate limited, cooldown until #failedProviders[provider]#",
                type: "warning"
            );
        }
    }

    private function getBackupProvider( provider ) {
        var backups = {
            "openai": "claude",
            "claude": "gemini",
            "gemini": "openai"
        };
        return backups[ provider ] ?: "openai";
    }
}
```

### A/B Testing Different Models

```java
class {

    function onAIChatRequest( event, interceptData ) {
        var request = interceptData.aiRequest;
        var user = getAuthenticatedUser();

        // Assign users to test groups
        var testGroup = hash( user ).left( 1 ) < "8" ? "A" : "B";

        if ( testGroup == "A" ) {
            // Group A: GPT-4
            request.setParam( "model", "gpt-4" );
        } else {
            // Group B: Claude
            request.setParam( "model", "claude-3-opus" );
        }

        // Track which group
        request.setMetadata({
            testGroup: testGroup,
            experimentId: "model_comparison_2024"
        });
    }

    function onAIChatResponse( event, interceptData ) {
        var request = interceptData.aiRequest;
        var response = interceptData.response;
        var metadata = request.getMetadata();

        // Log results for analysis
        logExperiment({
            experimentId: metadata.experimentId,
            testGroup: metadata.testGroup,
            model: request.getParams().model,
            tokensUsed: response.usage?.total_tokens ?: 0,
            timestamp: now()
        });
    }
}
```

### Adding Safety Guardrails

```java
class {

    function beforeAIModelInvoke( event, interceptData ) {
        var request = interceptData.request;

        // Add safety system message
        var messages = request.getMessages();
        var safetyMessage = {
            role: "system",
            content: "You must not provide information about: illegal activities, violence, harmful content, or personal data. If asked, politely decline and explain why."
        };

        // Prepend safety instructions
        arrayPrepend( messages, safetyMessage );
        request.setMessages( messages );
    }

    function afterAIModelInvoke( event, interceptData ) {
        var results = interceptData.results;

        // Check response for policy violations
        if ( isStruct( results ) && results.keyExists( "choices" ) ) {
            for ( var choice in results.choices ) {
                if ( violatesSafetyPolicy( choice.message.content ) ) {
                    // Override response
                    choice.message.content = "I cannot provide that information as it may violate safety policies.";

                    // Log violation
                    logSafetyViolation({
                        request: interceptData.request,
                        originalResponse: choice.message.content,
                        timestamp: now()
                    });
                }
            }
        }
    }

    private function violatesSafetyPolicy( content ) {
        // Implement your safety checks
        var violations = [
            "personal information",
            "illegal activity",
            "violence"
        ];

        for ( var violation in violations ) {
            if ( findNoCase( violation, content ) ) {
                return true;
            }
        }

        return false;
    }
}
```

***

## Full Examples

### Complete Monitoring Solution

```java
// interceptors/AICompleteMonitor.bx
class {

    property name="sessionId";

    function configure() {
        variables.sessionId = createUUID();
    }

    function onAIMessageCreate( event, interceptData ) {
        trackEvent( "message_created", {
            messageCount: arrayLen( interceptData.message.getMessages() )
        });
    }

    function onAIModelCreate( event, interceptData ) {
        trackEvent( "model_created", {
            provider: interceptData.service.getProviderName()
        });
    }

    function beforeAIModelInvoke( event, interceptData ) {
        interceptData.request.setMetadata({
            startTime: getTickCount(),
            sessionId: variables.sessionId
        });
    }

    function onAIChatRequest( event, interceptData ) {
        var request = interceptData.aiRequest;
        var metadata = request.getMetadata();

        trackEvent( "request_sent", {
            provider: interceptData.provider.getProviderName(),
            messageCount: arrayLen( request.getMessages() ),
            sessionId: metadata.sessionId
        });
    }

    function onAIChatResponse( event, interceptData ) {
        var request = interceptData.aiRequest;
        var response = interceptData.response;

        trackEvent( "response_received", {
            provider: interceptData.provider.getProviderName(),
            tokensUsed: response.usage?.total_tokens ?: 0,
            sessionId: request.getMetadata().sessionId
        });
    }

    function afterAIModelInvoke( event, interceptData ) {
        var request = interceptData.request;
        var metadata = request.getMetadata();
        var duration = getTickCount() - metadata.startTime;

        trackEvent( "invocation_complete", {
            duration: duration,
            sessionId: metadata.sessionId
        });
    }

    private function trackEvent( eventName, data ) {
        // Your analytics implementation
        analyticsService.track( eventName, data );
    }
}
```

### Security and Compliance

```java
// interceptors/AISecurityCompliance.bx
class {

    function onAIChatRequest( event, interceptData ) {
        var request = interceptData.aiRequest;
        var user = getAuthenticatedUser();

        // Validate user has permission
        if ( !hasPermission( user, "ai.use" ) ) {
            throw(
                type: "SecurityViolation",
                message: "User not authorized for AI operations"
            );
        }

        // Redact sensitive data
        var messages = request.getMessages();
        for ( var msg in messages ) {
            msg.content = redactSensitiveData( msg.content );
        }
        request.setMessages( messages );

        // Add audit trail
        request.setMetadata({
            userId: user.getId(),
            ipAddress: getClientIP(),
            timestamp: now()
        });
    }

    function onAIChatResponse( event, interceptData ) {
        // Log for compliance
        auditLog({
            userId: interceptData.aiRequest.getMetadata().userId,
            action: "ai_query",
            provider: interceptData.provider.getProviderName(),
            timestamp: now(),
            tokensUsed: interceptData.response.usage?.total_tokens ?: 0
        });
    }

    private function redactSensitiveData( text ) {
        // Email addresses
        text = reReplace( text, "\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b", "[EMAIL]", "all" );
        // Phone numbers
        text = reReplace( text, "\b\d{3}[-.]?\d{3}[-.]?\d{4}\b", "[PHONE]", "all" );
        // SSN
        text = reReplace( text, "\b\d{3}-\d{2}-\d{4}\b", "[SSN]", "all" );
        return text;
    }
}
```
