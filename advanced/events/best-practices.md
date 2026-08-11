---
description: >-
  Guidance for writing lightweight, safe, well-ordered, and testable event
  interceptors.
icon: circle-check
---

# Event Best Practices

### Keep Event Handlers Lightweight

Event handlers are called frequently. Keep processing minimal:

```java
// ❌ Bad: Heavy processing
function onAIChatRequest( event, interceptData ) {
    // This runs complex queries and blocks
    var history = getAllUserAIHistory( getUser() );
    analyzeHistoryForPatterns( history );
}

// ✅ Good: Lightweight, async if needed
function onAIChatRequest( event, interceptData ) {
    // Quick validation only
    validateRequest( interceptData.aiRequest );

    // Heavy work done async
    runAsync( function() {
        trackRequest( interceptData.aiRequest );
    });
}
```

### Handle Errors Gracefully

Don't let interceptor errors break AI operations:

```java
function onAIChatResponse( event, interceptData ) {
    try {
        // Your processing
        processResponse( interceptData.response );
    } catch ( any e ) {
        // Log but don't throw
        writeLog(
            text: "Error in interceptor: #e.message#",
            type: "error"
        );
        // Continue without breaking the flow
    }
}
```

### Document Side Effects

Make it clear what your interceptors modify:

```java
/**
 * AI Request Interceptor
 *
 * Modifies:
 * - Adds X-Request-ID header
 * - Overrides temperature to 0.7 if not set
 * - Logs request to database
 *
 * Does NOT modify:
 * - Message content
 * - Model selection
 */
function onAIChatRequest( event, interceptData ) {
    // Implementation
}
```

### Use Naming Conventions

```java
// Prefix interceptor classes with purpose
AIMonitoringInterceptor.bx
AISecurityInterceptor.bx
AICostTrackingInterceptor.bx
AIContentFilterInterceptor.bx
```

### Order Matters

Interceptors execute in registration order. Be mindful:

```java
interceptors = [
    { class: "SecurityInterceptor" },      // Run first: validate
    { class: "CostTrackingInterceptor" },  // Then: track costs
    { class: "LoggingInterceptor" }        // Finally: log
];
```

### Test Interceptors Independently

Write unit tests for your interceptor logic:

```java
// tests/interceptors/AIMonitorTest.bx
class extends="testbox.system.BaseSpec" {

    function run() {
        describe( "AIMonitor Interceptor", function() {

            it( "should log requests", function() {
                var monitor = new interceptors.AIMonitor();
                var interceptData = {
                    aiRequest: mockRequest(),
                    provider: mockProvider()
                };

                monitor.onAIChatRequest( {}, interceptData );

                // Verify logging occurred
                expect( getLogEntries() ).toHaveLength( 1 );
            });
        });
    }
}
```

### Make Interceptors Configurable

```java
class {

    property name="enabled" default=true;
    property name="logLevel" default="info";
    property name="destinations" default=["console","file"];

    function configure() {
        // Read from settings
        variables.enabled = getSetting( "monitoring.enabled" );
        variables.logLevel = getSetting( "monitoring.logLevel" );
    }

    function onAIChatRequest( event, interceptData ) {
        if ( !variables.enabled ) return;

        // Use configuration
        log( interceptData, variables.logLevel );
    }
}
```
