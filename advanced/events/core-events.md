---
description: >-
  Core lifecycle events for messages, chat requests, models, providers,
  transformers, tools, and pipelines.
icon: list-check
---

# Core Events

These are the fundamental events fired during object creation, model invocation, tool execution, pipeline runs, and error handling — the events most interceptors listen for.

### onAIMessageCreate

Fired when an AI message object is created via `aiMessage()`.

**When**: Message template creation **Frequency**: Once per `aiMessage()` call

#### Event Arguments

| Argument  | Type        | Description                |
| --------- | ----------- | -------------------------- |
| `message` | `AiMessage` | The created message object |

#### Example

```java
function onAIMessageCreate( event, interceptData ) {
    var message = interceptData.message;

    // Log message creation
    writeLog(
        text: "AI Message created with #arrayLen( message.getMessages() )# messages",
        type: "info"
    );

    // Add default system message if none exists
    var messages = message.getMessages();
    if ( messages.isEmpty() || messages[1].role != "system" ) {
        message.system( "You are a helpful assistant" );
    }
}
```

***

### onAIChatRequestCreate

Fired when an AI chat request object is created via `aiChatRequest()`.

**When**: Chat request object instantiation **Frequency**: Once per `aiChatRequest()` call

#### Event Arguments

| Argument    | Type           | Description                |
| ----------- | ----------- | -------------------------- |
| `aiRequest` | `AiChatRequest` | The created request object |

#### Example

```java
function onAIChatRequestCreate( event, interceptData ) {
    var request = interceptData.aiRequest;

    // Add tracking metadata
    request.setParams({
        user: getAuthenticatedUser(),
        requestId: createUUID(),
        timestamp: now()
    });

    // Apply organization-wide defaults
    if ( !request.getParams().keyExists( "temperature" ) ) {
        request.setParam( "temperature", 0.7 );
    }
}
```

***

### onAIProviderCreate

Fired when a provider/service instance is created.

**When**: After provider instantiation **Frequency**: Once per unique provider instance

#### Event Arguments

| Argument   | Type       | Description                  |
| ---------- | ---------- | ----------------------------- |
| `provider` | `IService` | The created service instance |

#### Example

```java
function onAIProviderCreate( event, interceptData ) {
    var service = interceptData.provider;

    // Configure provider-specific settings
    service.setTimeout( 60 );

    // Add custom headers
    service.setHeaders({
        "X-App-Version": getAppVersion(),
        "X-Environment": getEnvironment()
    });

    writeLog(
        text: "AI Provider created: #service.getProviderName()#",
        type: "info"
    );
}
```

***

### onAIModelCreate

Fired when an AI model runnable is created via `aiModel()`.

**When**: Model wrapper creation **Frequency**: Once per `aiModel()` call

#### Event Arguments

| Argument  | Type       | Description                |
| --------- | ---------- | -------------------------- |
| `model`   | `AiModel`  | The created model runnable |
| `service` | `IService` | The underlying service     |

#### Example

```java
function onAIModelCreate( event, interceptData ) {
    var model = interceptData.model;
    var service = interceptData.service;

    // Set default model parameters
    model.setParams({
        temperature: 0.7,
        max_tokens: 2000
    });

    // Track model creation
    logMetric( "ai.model.created", {
        provider: service.getProviderName(),
        timestamp: now()
    });
}
```

***

### onAITransformerCreate

Fired when a transform runnable is created via `aiTransform()`.

**When**: Transform function creation **Frequency**: Once per `aiTransform()` call

#### Event Arguments

| Argument    | Type                  | Description                    |
| ----------- | --------------------- | ------------------------------- |
| `transform` | `AiTransformRunnable` | The created transform runnable |

#### Example

```java
function onAITransformerCreate( event, interceptData ) {
    var transform = interceptData.transform;

    // Wrap transform with error handling
    var originalFn = transform.getTransformFn();
    transform.setTransformFn( function( input, params ) {
        try {
            return originalFn( input, params );
        } catch ( any e ) {
            logError( "Transform error: #e.message#" );
            return { error: true, message: e.message };
        }
    });
}
```

***

### beforeAIModelInvoke

Fired before an AI model is invoked (before sending to provider).

**When**: Before model execution **Frequency**: Every model invocation

#### Event Arguments

| Argument  | Type        | Description             |
| --------- | ----------- | ----------------------- |
| `model`   | `AiModel`   | The model being invoked |
| `request` | `AiChatRequest` | The request being sent  |

#### Example

```java
function beforeAIModelInvoke( event, interceptData ) {
    var model = interceptData.model;
    var request = interceptData.request;

    // Validate request
    if ( arrayLen( request.getMessages() ) == 0 ) {
        throw( "Cannot invoke model with empty messages" );
    }

    // Add request tracking
    request.setMetadata({
        invokeTimestamp: now(),
        userId: getAuthenticatedUser()
    });

    // Cost estimation
    var estimatedTokens = estimateTokenCount( request.getMessages() );
    writeLog(
        text: "Model invocation: ~#estimatedTokens# tokens",
        type: "info"
    );
}
```

***

### onAIChatRequest

Fired immediately before sending the HTTP request to the AI provider for chat operations.

**When**: Before chat HTTP request **Frequency**: Every chat API call (including streaming)

#### Event Arguments

| Argument     | Type        | Description                    |
| ------------ | ----------- | ------------------------------- |
| `dataPacket` | `Struct`    | The HTTP request data packet   |
| `aiRequest`  | `AiChatRequest` | The AI request object          |
| `provider`   | `IService`  | The service making the request |

#### Example

```java
function onAIChatRequest( event, interceptData ) {
    var dataPacket = interceptData.dataPacket;
    var request = interceptData.aiRequest;
    var provider = interceptData.provider;

    // Modify request before sending
    dataPacket.headers[ "X-Request-ID" ] = createUUID();

    // Log request details
    writeLog(
        text: "AI Request to #provider.getProviderName()#: " &
              "#arrayLen( request.getMessages() )# messages",
        type: "info",
        log: "ai-requests"
    );

    // Track costs
    trackAPICall( provider.getProviderName(), request.getParams() );

    // Add custom authentication
    if ( getSetting( "useCustomAuth" ) ) {
        dataPacket.headers[ "Authorization" ] = getCustomAuthToken();
    }
}
```

***

### onAIChatResponse

Fired after receiving the HTTP response from the AI provider for chat operations.

**When**: After chat HTTP response **Frequency**: Every chat API call (including streaming)

#### Event Arguments

| Argument      | Type        | Description                       |
| ------------- | ----------- | ---------------------------------- |
| `aiRequest`   | `AiChatRequest` | The original request              |
| `response`    | `Struct`    | The deserialized response         |
| `rawResponse` | `Struct`    | The raw HTTP response             |
| `provider`    | `IService`  | The service that made the request |

#### Example

```java
function onAIChatResponse( event, interceptData ) {
    var response = interceptData.response;
    var request = interceptData.aiRequest;
    var provider = interceptData.provider;

    // Extract usage information
    if ( response.keyExists( "usage" ) ) {
        var usage = response.usage;
        logUsage({
            provider: provider.getProviderName(),
            promptTokens: usage.prompt_tokens ?: 0,
            completionTokens: usage.completion_tokens ?: 0,
            totalTokens: usage.total_tokens ?: 0,
            timestamp: now()
        });
    }

    // Modify response
    if ( response.keyExists( "choices" ) && arrayLen( response.choices ) > 0 ) {
        // Add metadata to response
        response._metadata = {
            processedAt: now(),
            provider: provider.getProviderName(),
            cached: false
        };
    }

    // Cache response
    if ( getSetting( "cacheResponses" ) ) {
        cacheResponse( request, response );
    }
}
```

***

### afterAIModelInvoke

Fired after an AI model completes its invocation.

**When**: After model execution completes **Frequency**: Every model invocation

#### Event Arguments

| Argument  | Type        | Description                       |
| --------- | ----------- | ---------------------------------- |
| `model`   | `AiModel`   | The model that was invoked        |
| `request` | `AiChatRequest` | The request that was sent         |
| `results` | `Any`       | The results returned by the model |

#### Example

```java
function afterAIModelInvoke( event, interceptData ) {
    var model = interceptData.model;
    var request = interceptData.request;
    var results = interceptData.results;

    // Calculate execution time
    var startTime = request.getMetadata().invokeTimestamp ?: now();
    var duration = dateDiff( "s", startTime, now() );

    // Log completion
    writeLog(
        text: "Model invocation completed in #duration#s",
        type: "info"
    );

    // Track metrics
    recordMetric( "ai.model.duration", duration );

    // Validate response
    if ( isStruct( results ) && results.keyExists( "error" ) ) {
        logError( "Model returned error: #results.error.message#" );
    }
}
```

***

### onAIToolCreate

Fired when an AI tool is created via `aiTool()`.

**When**: Tool creation **Frequency**: Once per `aiTool()` call

#### Event Arguments

| Argument      | Type     | Description               |
| ------------- | -------- | -------------------------- |
| `tool`        | `Tool`   | The created tool instance |
| `name`        | `String` | Tool name                 |
| `description` | `String` | Tool description          |

#### Example

```java
function onAIToolCreate( event, interceptData ) {
    var tool = interceptData.tool;

    // Register tool in catalog
    registerToolInCatalog( tool.getName(), tool.getSchema() );

    // Validate tool configuration
    if ( !tool.getSchema().function.parameters.properties.isEmpty() ) {
        writeLog(
            text: "Tool created: #tool.getName()# with #structCount( tool.getSchema().function.parameters.properties )# parameters",
            type: "info"
        );
    }
}
```

***

### beforeAIToolExecute

Fired immediately before a tool's callable function is executed.

**When**: Before tool execution **Frequency**: Every tool call

#### Event Arguments

| Argument    | Type     | Description                  |
| ----------- | -------- | ------------------------------ |
| `tool`      | `Tool`   | The tool being executed      |
| `name`      | `String` | Tool name                    |
| `arguments` | `Struct` | Arguments passed to the tool |

#### Example

```java
function beforeAIToolExecute( event, interceptData ) {
    var tool = interceptData.tool;
    var args = interceptData.arguments;

    // Validate tool arguments
    validateToolArguments( tool.getName(), args );

    // Check permissions
    var user = getAuthenticatedUser();
    if ( !hasToolPermission( user, tool.getName() ) ) {
        throw( "User not authorized to execute tool: #tool.getName()#" );
    }

    // Rate limiting per tool
    if ( isToolRateLimited( tool.getName() ) ) {
        throw( "Tool rate limit exceeded: #tool.getName()#" );
    }

    // Log execution attempt
    writeLog(
        text: "Executing tool: #tool.getName()# with args: #serializeJSON(args)#",
        type: "info",
        log: "ai-tools"
    );
}
```

***

### afterAIToolExecute

Fired immediately after a tool's callable function completes execution.

**When**: After tool execution **Frequency**: Every tool call

#### Event Arguments

| Argument        | Type      | Description                    |
| --------------- | --------- | -------------------------------- |
| `tool`          | `Tool`    | The tool that was executed     |
| `name`          | `String`  | Tool name                      |
| `arguments`     | `Struct`  | Arguments passed to the tool   |
| `results`       | `Any`     | Results returned by the tool   |
| `executionTime` | `Numeric` | Execution time in milliseconds |

#### Example

```java
function afterAIToolExecute( event, interceptData ) {
    var tool = interceptData.tool;
    var results = interceptData.results;
    var executionTime = interceptData.executionTime;

    // Log execution metrics
    logMetric( "ai.tool.execution", {
        tool: tool.getName(),
        duration: executionTime,
        success: !isNull( results ),
        timestamp: now()
    });

    // Track tool usage
    trackToolUsage( tool.getName(), executionTime );

    // Alert on slow tools
    if ( executionTime > 5000 ) {
        writeLog(
            text: "Slow tool execution: #tool.getName()# took #executionTime#ms",
            type: "warning"
        );
    }

    // Validate results
    if ( isNull( results ) || results == "" ) {
        writeLog(
            text: "Tool returned empty result: #tool.getName()#",
            type: "warning"
        );
    }
}
```

***

### onAIError

Fired when an error occurs during AI operations (chat, embeddings, or streaming).

**When**: Before throwing provider errors **Frequency**: Every error condition

#### Event Arguments

| Argument           | Type                 | Description                                      |
| ------------------- | -------------------- | -------------------------------------------------- |
| `error`            | `Any`                | The error object/message from provider           |
| `errorMessage`     | `String`             | Formatted error message                          |
| `provider`         | `IService`           | The provider where error occurred                |
| `operation`        | `String`             | Operation type: "chat", "embeddings", "stream"   |
| `aiRequest`        | `AiChatRequest`          | The request that caused the error (if available) |
| `embeddingRequest` | `AiEmbeddingRequest` | For embedding errors                             |
| `canRetry`         | `Boolean`            | Whether operation can be retried                 |

#### Example

```java
class {

    property name="retryAttempts" default={};
    property name="maxRetries" default=3;

    function onAIError( event, interceptData ) {
        var error = interceptData.error;
        var provider = interceptData.provider.getProviderName();
        var operation = interceptData.operation;
        var canRetry = interceptData.canRetry;

        // Log the error
        writeLog(
            text: "AI Error in #provider# (#operation#): #interceptData.errorMessage#",
            type: "error",
            log: "ai-errors"
        );

        // Track error metrics
        recordMetric( "ai.error", {
            provider: provider,
            operation: operation,
            errorType: isStruct( error ) ? error.type : "unknown",
            timestamp: now()
        });

        // Implement retry logic
        if ( canRetry ) {
            var requestId = interceptData.aiRequest?.getMetadata().requestId ?: createUUID();
            var attempts = retryAttempts.keyExists( requestId ) ? retryAttempts[ requestId ] : 0;

            if ( attempts < variables.maxRetries ) {
                retryAttempts[ requestId ] = attempts + 1;

                writeLog(
                    text: "Retry attempt #attempts+1# of #maxRetries# for request #requestId#",
                    type: "warning"
                );

                // Wait before retry (exponential backoff)
                sleep( 1000 * ( 2 ^ attempts ) );

                // Signal to retry (implementation specific)
                interceptData.shouldRetry = true;
            } else {
                // Max retries exceeded
                writeLog(
                    text: "Max retries exceeded for request #requestId#",
                    type: "error"
                );

                // Cleanup retry tracking
                structDelete( retryAttempts, requestId );
            }
        }

        // Send alerts for critical errors
        if ( !canRetry || attempts >= maxRetries ) {
            sendErrorAlert({
                provider: provider,
                operation: operation,
                error: interceptData.errorMessage,
                timestamp: now()
            });
        }
    }
}
```

***

### onAIRateLimitHit

Fired when a provider returns a 429 (rate limit) HTTP status code.

**When**: When rate limit is detected **Frequency**: Every rate limit response

#### Event Arguments

| Argument     | Type        | Description                           |
| ------------ | ----------- | ---------------------------------------- |
| `provider`   | `IService`  | The provider that hit rate limit      |
| `operation`  | `String`    | Operation type: "chat", "embeddings"  |
| `statusCode` | `String`    | HTTP status code (429)                |
| `errorData`  | `Struct`    | Error response from provider          |
| `aiRequest`  | `AiChatRequest` | The request that hit the limit        |
| `retryAfter` | `String`    | Retry-After header value (if present) |

#### Example

```java
class {

    property name="rateLimitCooldowns" default={};

    function onAIRateLimitHit( event, interceptData ) {
        var provider = interceptData.provider.getProviderName();
        var retryAfter = interceptData.retryAfter;

        // Parse retry-after header (seconds or HTTP date)
        var cooldownSeconds = val( retryAfter );
        if ( cooldownSeconds == 0 && len( retryAfter ) ) {
            // Try parsing as HTTP date
            try {
                var retryDate = parseDateTime( retryAfter );
                cooldownSeconds = dateDiff( "s", now(), retryDate );
            } catch ( any e ) {
                cooldownSeconds = 60; // Default 1 minute
            }
        } else if ( cooldownSeconds == 0 ) {
            cooldownSeconds = 60; // Default if no header
        }

        var cooldownUntil = dateAdd( "s", cooldownSeconds, now() );
        rateLimitCooldowns[ provider ] = cooldownUntil;

        // Log rate limit
        writeLog(
            text: "Rate limit hit for #provider#. Cooldown until #cooldownUntil# (#cooldownSeconds#s)",
            type: "warning",
            log: "ai-rate-limits"
        );

        // Track in metrics
        recordMetric( "ai.rate_limit_hit", {
            provider: provider,
            cooldownSeconds: cooldownSeconds,
            timestamp: now()
        });

        // Send alert
        sendSlackAlert({
            channel: "#ai-monitoring",
            message: "🚨 Rate limit hit for #provider#. Cooling down for #cooldownSeconds# seconds.",
            color: "warning"
        });

        // Switch to backup provider if available
        var backupProvider = getBackupProvider( provider );
        if ( backupProvider != "" ) {
            writeLog(
                text: "Switching to backup provider: #backupProvider#",
                type: "info"
            );

            // Modify request to use backup (implementation specific)
            interceptData.useBackupProvider = backupProvider;
        }

        // Update application-wide rate limit status
        application.aiRateLimits[ provider ] = {
            limitedUntil: cooldownUntil,
            backupProvider: backupProvider
        };
    }
}
```

***

### beforeAIPipelineRun

Fired before a runnable pipeline sequence begins execution.

**When**: Before pipeline execution starts **Frequency**: Every pipeline run

#### Event Arguments

| Argument    | Type                 | Description                   |
| ----------- | --------------------- | -------------------------------- |
| `sequence`  | `AiRunnableSequence` | The sequence being executed   |
| `name`      | `String`             | Sequence name                 |
| `stepCount` | `Numeric`            | Number of steps in pipeline   |
| `steps`     | `Array`              | Array of step information     |
| `input`     | `Any`                | Initial input to pipeline     |
| `params`    | `Struct`             | Parameters passed to pipeline |
| `options`   | `Struct`             | Options passed to pipeline    |

#### Example

```java
function beforeAIPipelineRun( event, interceptData ) {
    var sequence = interceptData.sequence;
    var stepCount = interceptData.stepCount;
    var steps = interceptData.steps;

    // Log pipeline execution start
    writeLog(
        text: "Starting pipeline: #sequence.getName()# with #stepCount# steps",
        type: "info",
        log: "ai-pipelines"
    );

    // Print pipeline structure for debugging
    for ( var step in steps ) {
        writeLog(
            text: "  Step #step.index#: #step.name# (#step.type#)",
            type: "info"
        );
    }

    // Validate pipeline configuration
    if ( stepCount == 0 ) {
        throw( "Cannot run empty pipeline" );
    }

    // Add tracking metadata
    var pipelineId = createUUID();
    interceptData.pipelineId = pipelineId;

    // Track pipeline execution
    recordPipelineStart({
        pipelineId: pipelineId,
        name: sequence.getName(),
        stepCount: stepCount,
        timestamp: now()
    });
}
```

***

### afterAIPipelineRun

Fired after a runnable pipeline sequence completes execution.

**When**: After pipeline execution completes **Frequency**: Every pipeline run

#### Event Arguments

| Argument        | Type                 | Description                          |
| ---------------- | --------------------- | ---------------------------------------- |
| `sequence`      | `AiRunnableSequence` | The sequence that was executed       |
| `name`          | `String`             | Sequence name                        |
| `stepCount`     | `Numeric`            | Number of steps in pipeline          |
| `steps`         | `Array`              | Array of step information            |
| `input`         | `Any`                | Initial input to pipeline            |
| `result`        | `Any`                | Final result from pipeline           |
| `executionTime` | `Numeric`            | Total execution time in milliseconds |

#### Example

```java
function afterAIPipelineRun( event, interceptData ) {
    var sequence = interceptData.sequence;
    var result = interceptData.result;
    var executionTime = interceptData.executionTime;
    var stepCount = interceptData.stepCount;

    // Log pipeline completion
    writeLog(
        text: "Pipeline completed: #sequence.getName()# in #executionTime#ms (#stepCount# steps)",
        type: "info",
        log: "ai-pipelines"
    );

    // Track metrics
    recordMetric( "ai.pipeline.duration", {
        name: sequence.getName(),
        duration: executionTime,
        stepCount: stepCount,
        timestamp: now()
    });

    // Alert on slow pipelines
    if ( executionTime > 10000 ) {
        writeLog(
            text: "Slow pipeline execution: #sequence.getName()# took #executionTime#ms",
            type: "warning"
        );
    }

    // Calculate average time per step
    var avgStepTime = executionTime / stepCount;
    writeLog(
        text: "Average time per step: #numberFormat(avgStepTime, '0.00')#ms",
        type: "info"
    );

    // Validate results
    if ( isNull( result ) ) {
        writeLog(
            text: "Pipeline returned null result: #sequence.getName()#",
            type: "warning"
        );
    }
}
```

***

### onAIToolRegistryRegister

Fired when a tool is registered with the `AIToolRegistry`.

**When**: After a tool is added via `aiToolRegistry().register()` or `scan()` **Frequency**: Once per registered tool

#### Event Arguments

| Argument | Type | Description |
| --- | --- | --- |
| `tool` | `ITool` | The tool instance that was registered |
| `key` | `string` | The registry key (e.g., `"search"` or `"search@my-module"`) |
| `module` | `string` | The module name (empty string if none) |

#### Example

```javascript
function onAIToolRegistryRegister( event, interceptData ) {
    writeLog(
        text: "Tool registered: #interceptData.key#",
        type: "info"
    );
}
```

***

### onAIToolRegistryUnregister

Fired when a tool is removed from the `AIToolRegistry`.

**When**: After a tool is removed via `aiToolRegistry().unregister()` or `unregisterByModule()` **Frequency**: Once per removed tool

#### Event Arguments

| Argument | Type | Description |
| --- | --- | --- |
| `key` | `string` | The registry key of the removed tool |
| `module` | `string` | The module name extracted from the key |

#### Example

```javascript
function onAIToolRegistryUnregister( event, interceptData ) {
    writeLog(
        text: "Tool unregistered: #interceptData.key#",
        type: "info"
    );
}
```
