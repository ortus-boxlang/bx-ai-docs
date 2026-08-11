---
description: >-
  Track token usage and cost per request, and attribute usage to tenants,
  departments, or cost centers.
icon: calculator
---

# Token & Usage Events

### onAITokenCount

Fired when token usage information is available from the AI provider response.

**When**: After receiving response with usage data **Frequency**: Every successful API call that returns usage

#### Event Arguments

| Argument           | Type            | Description                          |
| ------------------- | ---------------- | -------------------------------------- |
| `provider`         | `IService`      | The provider instance                |
| `operation`        | `String`        | Operation type: `"chat"`             |
| `model`            | `String`        | Model ID used for the request        |
| `promptTokens`     | `Numeric`       | Input tokens used                    |
| `completionTokens` | `Numeric`       | Output tokens generated              |
| `totalTokens`      | `Numeric`       | Total tokens (prompt + completion)   |
| `aiRequest`        | `AiChatRequest` | The originating chat request object  |
| `usage`            | `Struct`        | Full raw usage object from provider  |
| `timestamp`        | `DateTime`      | When the event was fired             |

#### Example

```java
class {

    property name="monthlyUsage" default={};
    property name="budgetLimits" default={
        "openai": 10000,     // $10
        "claude": 15000,     // $15
        "gemini": 20000      // $20
    };

    function onAITokenCount( event, interceptData ) {
        var provider = interceptData.provider.getProviderName();
        var model = interceptData.model;
        var totalTokens = interceptData.totalTokens;
        var promptTokens = interceptData.promptTokens;
        var completionTokens = interceptData.completionTokens;

        // Calculate cost based on provider pricing
        var cost = calculateTokenCost(
            provider,
            model,
            promptTokens,
            completionTokens
        );

        // Track monthly usage
        var currentMonth = dateFormat( now(), "yyyy-mm" );
        if ( !monthlyUsage.keyExists( currentMonth ) ) {
            monthlyUsage[ currentMonth ] = {};
        }
        if ( !monthlyUsage[ currentMonth ].keyExists( provider ) ) {
            monthlyUsage[ currentMonth ][ provider ] = {
                tokens: 0,
                cost: 0,
                requests: 0
            };
        }

        monthlyUsage[ currentMonth ][ provider ].tokens += totalTokens;
        monthlyUsage[ currentMonth ][ provider ].cost += cost;
        monthlyUsage[ currentMonth ][ provider ].requests++;

        // Log token usage
        writeLog(
            text: "Token usage: #provider# (#model#) - #totalTokens# tokens ($#numberFormat(cost, '0.0000')#)",
            type: "info",
            log: "ai-tokens"
        );

        // Store in database for analytics
        queryExecute(
            "INSERT INTO ai_token_usage (provider, model, prompt_tokens, completion_tokens, total_tokens, cost, created_at) VALUES (?, ?, ?, ?, ?, ?, ?)",
            [
                provider,
                model,
                promptTokens,
                completionTokens,
                totalTokens,
                cost,
                now()
            ]
        );

        // Check budget limits
        var currentCost = monthlyUsage[ currentMonth ][ provider ].cost;
        var budgetLimit = budgetLimits.keyExists( provider ) ? budgetLimits[ provider ] : 0;

        if ( budgetLimit > 0 ) {
            var percentUsed = ( currentCost / budgetLimit ) * 100;

            // Alert at 80% budget
            if ( percentUsed >= 80 && percentUsed < 100 ) {
                sendBudgetAlert({
                    provider: provider,
                    percentUsed: percentUsed,
                    currentCost: currentCost,
                    budgetLimit: budgetLimit,
                    level: "warning"
                });
            }

            // Block at 100% budget
            if ( currentCost >= budgetLimit ) {
                writeLog(
                    text: "Budget limit exceeded for #provider#: $#currentCost# >= $#budgetLimit#",
                    type: "error"
                );

                sendBudgetAlert({
                    provider: provider,
                    percentUsed: percentUsed,
                    currentCost: currentCost,
                    budgetLimit: budgetLimit,
                    level: "critical"
                });

                // Optionally throw to prevent further usage
                // throw( "Budget limit exceeded for provider: #provider#" );
            }
        }

        // Track cost per user
        var user = interceptData.aiRequest.getMetadata().userId ?: "anonymous";
        trackUserCost( user, provider, cost, totalTokens );
    }

    private function calculateTokenCost( provider, model, promptTokens, completionTokens ) {
        // Pricing per 1M tokens (as of 2024)
        var pricing = {
            "openai": {
                "gpt-4": { prompt: 30.00, completion: 60.00 },
                "gpt-4-turbo": { prompt: 10.00, completion: 30.00 },
                "gpt-3.5-turbo": { prompt: 0.50, completion: 1.50 }
            },
            "claude": {
                "claude-3-opus": { prompt: 15.00, completion: 75.00 },
                "claude-3-sonnet": { prompt: 3.00, completion: 15.00 }
            },
            "gemini": {
                "gemini-pro": { prompt: 0.50, completion: 1.50 }
            }
        };

        // Get pricing for model
        var modelPricing = {};
        if ( pricing.keyExists( provider ) ) {
            for ( var key in pricing[ provider ] ) {
                if ( findNoCase( key, model ) ) {
                    modelPricing = pricing[ provider ][ key ];
                    break;
                }
            }
        }

        // Default pricing if not found
        if ( modelPricing.isEmpty() ) {
            modelPricing = { prompt: 1.00, completion: 2.00 };
        }

        // Calculate cost (per 1M tokens)
        var promptCost = ( promptTokens / 1000000 ) * modelPricing.prompt;
        var completionCost = ( completionTokens / 1000000 ) * modelPricing.completion;

        return promptCost + completionCost;
    }
}
```

***

#### Multi-Tenant Usage Tracking (v2.1.0+)

Track AI usage per tenant for billing and cost allocation. Works with **aiChat()**, **aiChatAsync()**, **aiChatStream()**, and **AI Agent runnables**.

```javascript
// Set tenant context in AI chat requests
result = aiChat(
    messages: "Analyze this data",
    options: {
        tenantId: "customer_acme",
        usageMetadata: {
            costCenter: "engineering",
            projectId: "proj-2024-ml",
            userId: "john@acme.com",
            department: "R&D"
        }
    }
)

// Set tenant context in AI agents
agent = aiAgent(
    name: "Assistant",
    memory: aiMemory( "window" )
)

result = agent.run(
    input: "Help me with this task",
    options: {
        tenantId: "customer_acme",
        usageMetadata: {
            costCenter: "support",
            ticketId: "TICK-12345",
            priority: "high"
        }
    }
)

// Set tenant context in model runnables
model = aiModel( provider: "openai" )

result = model.run(
    input: "Generate a report",
    params: { temperature: 0.7 },
    options: {
        tenantId: "org_finance",
        usageMetadata: {
            department: "accounting",
            reportType: "quarterly"
        }
    }
)
```

**Interceptor for Multi-Tenant Billing:**

```javascript
class {

    property name="billingService" inject="BillingService";

    function onAITokenCount( event, interceptData ) {
        // Extract tenant context
        var tenantId = interceptData.tenantId ?: "unknown";
        var usageMetadata = interceptData.usageMetadata ?: {};

        // Provider and model info
        var provider = interceptData.provider.getProviderName();
        var model = interceptData.model;

        // Token usage
        var totalTokens = interceptData.totalTokens;
        var promptTokens = interceptData.promptTokens;
        var completionTokens = interceptData.completionTokens;

        // Calculate cost
        var cost = calculateCost(
            provider: provider,
            model: model,
            promptTokens: promptTokens,
            completionTokens: completionTokens
        );

        // Store usage for tenant billing
        billingService.recordUsage({
            tenantId: tenantId,
            provider: provider,
            model: model,
            operation: interceptData.operation,
            totalTokens: totalTokens,
            promptTokens: promptTokens,
            completionTokens: completionTokens,
            estimatedCost: cost,
            timestamp: interceptData.timestamp,
            metadata: usageMetadata
        });

        // Check tenant quota
        var tenantQuota = billingService.getTenantQuota( tenantId );
        var currentUsage = billingService.getCurrentMonthUsage( tenantId );

        if ( currentUsage.cost + cost > tenantQuota.limit ) {
            // Send alert
            emailService.send(
                to: tenantQuota.alertEmail,
                subject: "AI Usage Quota Alert: #tenantId#",
                body: "Current usage: $#currentUsage.cost# + $#cost# exceeds limit: $#tenantQuota.limit#"
            );

            // Optionally block further usage
            if ( tenantQuota.hardLimit ) {
                throw(
                    type: "QuotaExceeded",
                    message: "Tenant #tenantId# has exceeded their AI usage quota"
                );
            }
        }

        // Log for analytics
        writeLog(
            type: "info",
            file: "ai-usage",
            text: "Tenant=#tenantId#, Provider=#provider#, Model=#model#, Tokens=#totalTokens#, Cost=$#numberFormat(cost,'0.0000')#, Metadata=#serializeJSON(usageMetadata)#"
        );
    }
}
```

**Multi-Provider Tracking Example:**

```javascript
// Track usage across different providers
class TenantUsageTracker {

    property name="usageDB" inject="UsageDatabase";

    function onAITokenCount( event, interceptData ) {
        var tenantId = interceptData.tenantId;

        // Skip if no tenant context
        if ( isNull( tenantId ) || tenantId == "" ) return;

        var usageRecord = {
            tenantId: tenantId,
            provider: interceptData.provider.getProviderName(),
            model: interceptData.model,
            operation: interceptData.operation,
            tokens: {
                prompt: interceptData.promptTokens,
                completion: interceptData.completionTokens,
                total: interceptData.totalTokens
            },
            cost: calculateProviderCost( interceptData ),
            timestamp: interceptData.timestamp,
            metadata: interceptData.usageMetadata ?: {},
            providerOptions: interceptData.providerOptions ?: {}
        };

        // Store in database
        usageDB.insertUsage( usageRecord );

        // Update real-time metrics
        metricsService.increment( "ai.usage.#tenantId#.tokens", usageRecord.tokens.total );
        metricsService.increment( "ai.usage.#tenantId#.requests", 1 );
        metricsService.gauge( "ai.usage.#tenantId#.cost", usageRecord.cost );

        // Track by cost center if provided
        if ( structKeyExists( usageRecord.metadata, "costCenter" ) ) {
            var costCenter = usageRecord.metadata.costCenter;
            metricsService.increment(
                "ai.usage.#tenantId#.#costCenter#.cost",
                usageRecord.cost
            );
        }
    }

    private function calculateProviderCost( data ) {
        // Provider-specific pricing (per 1M tokens)
        var pricing = {
            "openai": {
                "gpt-4o": { prompt: 2.50, completion: 10.00 },
                "gpt-4o-mini": { prompt: 0.150, completion: 0.600 }
            },
            "bedrock": {
                "claude-3-sonnet": { prompt: 3.00, completion: 15.00 },
                "claude-3-haiku": { prompt: 0.25, completion: 1.25 }
            },
            "ollama": {
                // Free for local models
                "*": { prompt: 0, completion: 0 }
            },
            "deepseek": {
                "deepseek-chat": { prompt: 0.14, completion: 0.28 }
            }
        };

        var provider = data.provider.getProviderName();
        var model = data.model;

        // Get pricing for this provider/model
        var modelPricing = { prompt: 0, completion: 0 };

        if ( structKeyExists( pricing, provider ) ) {
            // Try exact model match
            if ( structKeyExists( pricing[ provider ], model ) ) {
                modelPricing = pricing[ provider ][ model ];
            } else {
                // Try wildcard match
                for ( var pattern in pricing[ provider ] ) {
                    if ( pattern == "*" || findNoCase( pattern, model ) ) {
                        modelPricing = pricing[ provider ][ pattern ];
                        break;
                    }
                }
            }
        }

        // Calculate cost
        var promptCost = ( data.promptTokens / 1000000 ) * modelPricing.prompt;
        var completionCost = ( data.completionTokens / 1000000 ) * modelPricing.completion;

        return promptCost + completionCost;
    }
}
```

**Usage Metadata Examples:**

```javascript
// Example 1: Department-level tracking
aiChat(
    messages: "Generate report",
    options: {
        tenantId: "company_xyz",
        usageMetadata: {
            department: "marketing",
            costCenter: "CC-4501",
            campaign: "spring-2026"
        }
    }
)

// Example 2: Project-based tracking
aiChat(
    messages: "Analyze customer data",
    options: {
        tenantId: "client_acme",
        usageMetadata: {
            projectId: "proj-analytics-2026",
            projectName: "Customer Insights",
            billableHours: true,
            clientPO: "PO-12345"
        }
    }
)

// Example 3: User-level tracking
aiChat(
    messages: "Help me with code",
    options: {
        tenantId: "org_developers",
        usageMetadata: {
            userId: "john.doe@company.com",
            team: "backend",
            feature: "api-optimization"
        }
    }
)

// Example 4: Geographic tracking
aiChat(
    messages: "Translate content",
    options: {
        tenantId: "saas_customer_456",
        usageMetadata: {
            region: "us-west",
            datacenter: "pdx-1",
            customerTier: "enterprise"
        }
    }
)
```

**Benefits of Multi-Tenant Tracking:**

* ✅ **Accurate Billing**: Attribute AI costs to specific tenants/customers
* ✅ **Cost Allocation**: Track usage by department, project, or cost center
* ✅ **Quota Management**: Enforce per-tenant usage limits
* ✅ **Analytics**: Understand which tenants/projects use AI most
* ✅ **Chargeback**: Generate detailed usage reports for internal billing
* ✅ **Provider Agnostic**: Works with all AI providers (OpenAI, Bedrock, Ollama, etc.)

****

### Event Priority Reference

Events fire in this order during a typical AI chat with tools:

1. `onAIMessageCreate` - Message template created
2. `onAIChatRequestCreate` - Request object created
3. `onAIModelCreate` - Model wrapper created
4. `onAIToolCreate` - Tool(s) created (if using tools)
5. `beforeAIPipelineRun` - Pipeline about to start (if using pipelines)
6. `beforeAIModelInvoke` - Model about to be invoked
7. `onAIChatRequest` - HTTP request about to be sent
8. `onAIRateLimitHit` - If rate limit encountered
9. `onAIChatResponse` - HTTP response received
10. `onAITokenCount` - Token usage tracked
11. `beforeAIToolExecute` - Tool about to execute (if AI requested tool call)
12. `afterAIToolExecute` - Tool execution complete
13. `onAIError` - If any error occurred
14. `afterAIModelInvoke` - Model invocation complete
15. `afterAIPipelineRun` - Pipeline execution complete
