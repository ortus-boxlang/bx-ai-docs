---
description: >-
  Monitoring, observability, performance optimization, and cost management
  for production BoxLang AI deployments.
icon: chart-line
---

# Monitoring & Performance

## 📊 Monitoring & Observability

### Event-Based Monitoring

Use BoxLang AI's event system:

```javascript
// EventListener.bx
class {
    function onAIRequest( event, interceptData ) {
        var startTime = getTickCount()
        interceptData.startTime = startTime

        writeLog(
            "AI Request: provider=#interceptData.provider#, model=#interceptData.chatRequest.model#",
            "info"
        )
    }

    function onAIResponse( event, interceptData ) {
        var duration = getTickCount() - interceptData.startTime
        var tokenCount = interceptData.response.usage?.total_tokens ?: 0

        // Log metrics
        writeLog(
            "AI Response: duration=#duration#ms, tokens=#tokenCount#, provider=#interceptData.provider#",
            "info"
        )

        // Send to monitoring
        sendMetric( "ai.request.duration", duration, {
            provider: interceptData.provider,
            model: interceptData.chatRequest.model
        } )

        sendMetric( "ai.tokens.used", tokenCount, {
            provider: interceptData.provider
        } )

        // Track costs
        var cost = estimateCost( interceptData.provider, tokenCount )
        sendMetric( "ai.cost", cost, {
            provider: interceptData.provider
        } )
    }

    function onAIError( event, interceptData ) {
        writeLog(
            "AI Error: #interceptData.error.message#, provider=#interceptData.provider#",
            "error"
        )

        sendMetric( "ai.errors", 1, {
            provider: interceptData.provider,
            errorType: interceptData.error.type
        } )

        // Alert on high error rate
        if ( getErrorRate() > 0.05 ) {  // 5% error threshold
            notifyOps( "High AI error rate detected" )
        }
    }
}

// Register listeners in ModuleConfig.bx or Application.bx
BoxRegisterInterceptor(
    interceptorObject: new EventListener(),
    interceptorName: "AIMonitoring"
)
```

### Metrics Collection

```javascript
class singleton {
    property name="metrics" type="struct";

    function init() {
        variables.metrics = {
            requests: 0,
            successes: 0,
            failures: 0,
            totalTokens: 0,
            totalCost: 0,
            byProvider: {}
        }
        return this
    }

    function recordRequest(
        required string provider,
        required numeric tokens,
        required numeric cost,
        boolean success = true
    ) {
        lock name="metrics" type="exclusive" timeout="5" {
            variables.metrics.requests++

            if ( arguments.success ) {
                variables.metrics.successes++
            } else {
                variables.metrics.failures++
            }

            variables.metrics.totalTokens += arguments.tokens
            variables.metrics.totalCost += arguments.cost

            if ( !structKeyExists( variables.metrics.byProvider, arguments.provider ) ) {
                variables.metrics.byProvider[ arguments.provider ] = {
                    requests: 0,
                    tokens: 0,
                    cost: 0
                }
            }

            variables.metrics.byProvider[ arguments.provider ].requests++
            variables.metrics.byProvider[ arguments.provider ].tokens += arguments.tokens
            variables.metrics.byProvider[ arguments.provider ].cost += arguments.cost
        }
    }

    function getMetrics() {
        lock name="metrics" type="readonly" timeout="5" {
            return duplicate( variables.metrics )
        }
    }

    function reset() {
        init()
    }
}

// Expose metrics endpoint
// /api/metrics
function metrics() {
    var metricsService = getInstance( "MetricsService" )
    return renderJSON( metricsService.getMetrics() )
}
```

### Health Checks

```javascript
// /health endpoint
function health() {
    var status = {
        status: "healthy",
        timestamp: now(),
        checks: {}
    }

    // Check AI providers
    providers = [ "openai", "claude" ]
    for ( provider in providers ) {
        try {
            var start = getTickCount()
            aiChat( "test", { max_tokens: 1 }, { provider: provider } )
            status.checks[ provider ] = {
                status: "up",
                responseTime: getTickCount() - start
            }
        } catch ( any e ) {
            status.status = "degraded"
            status.checks[ provider ] = {
                status: "down",
                error: e.message
            }
        }
    }

    // Check memory systems
    try {
        var memory = aiMemory( "cache" )
        memory.add( { role: "user", content: "test" } )
        memory.clear()
        status.checks.memory = { status: "up" }
    } catch ( any e ) {
        status.status = "degraded"
        status.checks.memory = {
            status: "down",
            error: e.message
        }
    }

    // Set HTTP status code
    if ( status.status == "healthy" ) {
        setHTTPStatus( 200 )
    } else {
        setHTTPStatus( 503 )
    }

    return renderJSON( status )
}
```

### Logging Best Practices

```javascript
// Structured logging
function logAIInteraction(
    required string action,
    required string provider,
    struct metadata = {}
) {
    var logData = {
        timestamp: now(),
        action: arguments.action,
        provider: arguments.provider,
        userId: session.userId ?: "anonymous",
        requestId: request.requestId ?: createUUID(),
        metadata: arguments.metadata
    }

    writeLog(
        text: serializeJSON( logData ),
        type: "information",
        file: "ai-interactions"
    )
}

// Usage
logAIInteraction(
    action: "chat_request",
    provider: "openai",
    metadata: {
        model: "gpt-4",
        promptLength: len( prompt ),
        temperature: 0.7
    }
)
```

***


## ⚡ Performance Optimization

### Response Caching

```javascript
class {
    function getCachedAIResponse(
        required string prompt,
        struct params = {},
        numeric ttl = 3600
    ) {
        // Generate cache key
        var cacheKey = hash(
            serializeJSON({
                prompt: arguments.prompt,
                params: arguments.params
            }),
            "MD5"
        )

        // Try cache first
        var cached = cacheGet( "ai_#cacheKey#" )
        if ( !isNull( cached ) ) {
            writeLog( "AI cache hit for: #left( arguments.prompt, 50 )#..." )
            return cached
        }

        // Call AI
        var response = aiChat( arguments.prompt, arguments.params )

        // Cache response
        cacheSet(
            "ai_#cacheKey#",
            response,
            arguments.ttl
        )

        return response
    }
}
```

### Connection Pooling

```javascript
// For JDBC memory
datasource = {
    name: "aiMemory",
    driver: "postgresql",
    url: "jdbc:postgresql://localhost:5432/ai_db",
    username: getSystemSetting( "DB_USER" ),
    password: getSystemSetting( "DB_PASSWORD" ),

    // Connection pool settings
    maxConnections: 50,
    minConnections: 10,
    maxIdleTime: 30,
    connectionTimeout: 5000,
    validationQuery: "SELECT 1"
}
```

### Async Processing

```javascript
// Non-blocking AI calls
function processUserRequest( required string prompt ) {
    // Return immediately with request ID
    var requestId = createUUID()

    // Process in background
    runAsync( () => {
        try {
            var response = aiChat( arguments.prompt )

            // Store result
            cacheSet( "ai_result_#requestId#", {
                status: "completed",
                response: response
            }, 300 )

            // Notify user (websocket, email, etc.)
            notifyUser( session.userId, requestId, response )

        } catch ( any e ) {
            cacheSet( "ai_result_#requestId#", {
                status: "failed",
                error: e.message
            }, 300 )
        }
    } )

    return {
        requestId: requestId,
        status: "processing"
    }
}

// Check result endpoint
function checkResult( required string requestId ) {
    var result = cacheGet( "ai_result_#arguments.requestId#" )

    if ( isNull( result ) ) {
        return { status: "processing" }
    }

    return result
}
```

### Batch Processing

```javascript
// Process multiple prompts efficiently
function batchAIChat( required array prompts ) {
    var futures = []

    // Start all requests in parallel
    for ( var prompt in arguments.prompts ) {
        futures.append(
            aiChatAsync( prompt )
        )
    }

    // Collect results
    var results = []
    for ( var future in futures ) {
        try {
            results.append({
                success: true,
                response: future.get( 30, "seconds" )
            })
        } catch ( any e ) {
            results.append({
                success: false,
                error: e.message
            })
        }
    }

    return results
}
```

***


## 💰 Cost Management

### Usage Tracking

```javascript
class singleton {
    property name="dailyUsage" type="struct";

    function init() {
        variables.dailyUsage = {}
        return this
    }

    function trackUsage(
        required string provider,
        required numeric tokens,
        required numeric cost
    ) {
        var today = dateFormat( now(), "yyyy-mm-dd" )

        lock name="usage_#today#" type="exclusive" timeout="5" {
            if ( !structKeyExists( variables.dailyUsage, today ) ) {
                variables.dailyUsage[ today ] = {
                    totalTokens: 0,
                    totalCost: 0,
                    byProvider: {}
                }
            }

            variables.dailyUsage[ today ].totalTokens += arguments.tokens
            variables.dailyUsage[ today ].totalCost += arguments.cost

            if ( !structKeyExists( variables.dailyUsage[ today ].byProvider, arguments.provider ) ) {
                variables.dailyUsage[ today ].byProvider[ arguments.provider ] = {
                    tokens: 0,
                    cost: 0
                }
            }

            variables.dailyUsage[ today ].byProvider[ arguments.provider ].tokens += arguments.tokens
            variables.dailyUsage[ today ].byProvider[ arguments.provider ].cost += arguments.cost
        }

        // Check budget limits
        checkBudget( today )
    }

    function checkBudget( required string date ) {
        var dailyBudget = getSystemSetting( "AI_DAILY_BUDGET", 100 )
        var usage = variables.dailyUsage[ arguments.date ]

        if ( usage.totalCost > dailyBudget ) {
            writeLog(
                "Daily AI budget exceeded: $#usage.totalCost# > $#dailyBudget#",
                "critical"
            )
            notifyOps( "AI budget alert", {
                date: arguments.date,
                spent: usage.totalCost,
                budget: dailyBudget
            } )
        } else if ( usage.totalCost > ( dailyBudget * 0.8 ) ) {
            writeLog(
                "AI budget at 80%: $#usage.totalCost# / $#dailyBudget#",
                "warning"
            )
        }
    }

    function getDailyUsage( string date ) {
        var targetDate = arguments.date ?: dateFormat( now(), "yyyy-mm-dd" )
        return variables.dailyUsage[ targetDate ] ?: {}
    }
}
```

### Cost Estimation

```javascript
function estimateCost(
    required string provider,
    required numeric inputTokens,
    required numeric outputTokens
) {
    // Pricing per 1M tokens (as of Dec 2024)
    var pricing = {
        "openai": {
            "gpt-4o": { input: 2.50, output: 10.00 },
            "gpt-4-turbo": { input: 10.00, output: 30.00 },
            "gpt-3.5-turbo": { input: 0.50, output: 1.50 }
        },
        "claude": {
            "claude-3-opus": { input: 15.00, output: 75.00 },
            "claude-3-sonnet": { input: 3.00, output: 15.00 },
            "claude-3-haiku": { input: 0.25, output: 1.25 }
        },
        "gemini": {
            "gemini-1.5-pro": { input: 1.25, output: 5.00 },
            "gemini-1.5-flash": { input: 0.075, output: 0.30 }
        }
    }

    var rates = pricing[ arguments.provider ][ model ] ?: { input: 0, output: 0 }

    var inputCost = ( arguments.inputTokens / 1000000 ) * rates.input
    var outputCost = ( arguments.outputTokens / 1000000 ) * rates.output

    return inputCost + outputCost
}
```

***

