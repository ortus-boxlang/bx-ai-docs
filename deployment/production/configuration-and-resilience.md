---
description: >-
  Configuration management and error handling & resilience patterns for
  production BoxLang AI deployments.
icon: gears
---

# Configuration & Resilience

## ⚙️ Configuration Management

### Environment-Based Configuration

**Separate configs per environment**:

```javascript
// config/ai-config.bx
class {
    function getAIConfig() {
        var env = getSystemSetting( "ENVIRONMENT", "production" )

        var configs = {
            "development": {
                provider: "ollama",           // Free for dev
                model: "llama3.2:3b",
                timeout: 30,
                retries: 1,
                logLevel: "DEBUG"
            },
            "staging": {
                provider: "openai",
                model: "gpt-3.5-turbo",       // Cheaper for staging
                timeout: 15,
                retries: 2,
                logLevel: "INFO"
            },
            "production": {
                provider: "openai",
                model: "gpt-4o",              // Best quality for prod
                fallbackProvider: "claude",
                timeout: 10,
                retries: 3,
                logLevel: "WARN",
                enableCaching: true,
                enableMonitoring: true
            }
        }

        return configs[ env ]
    }
}
```

### Secrets Management

**Never hardcode API keys**:

```javascript
// ❌ WRONG
apiKey = "sk-1234567890abcdef"

// ✅ RIGHT - Environment variables
apiKey = getSystemSetting( "OPENAI_API_KEY" )

// ✅ RIGHT - AWS Secrets Manager
function getAPIKey( secretName ) {
    return awsSecretsManager.getSecret( secretName ).getValue()
}

// ✅ RIGHT - Azure Key Vault
function getAPIKey( secretName ) {
    return azureKeyVault.getSecret( secretName )
}

// ✅ RIGHT - HashiCorp Vault
function getAPIKey( path ) {
    return vaultClient.read( path ).data.apiKey
}
```

### Dynamic Configuration Reloading

```javascript
// Reload config without restarting app
class singleton {
    property name="config" type="struct";
    property name="lastReload" type="date";

    function init() {
        reloadConfig()
        return this
    }

    function reloadConfig() {
        variables.config = deserializeJSON(
            fileRead( "/config/ai-production.json" )
        )
        variables.lastReload = now()
        writeLog( "AI configuration reloaded" )
    }

    function getConfig() {
        // Auto-reload every 5 minutes
        if ( dateDiff( "n", variables.lastReload, now() ) > 5 ) {
            reloadConfig()
        }
        return variables.config
    }
}
```

***


## 🛡️ Error Handling & Resilience

### Comprehensive Error Handling

```javascript
class {
    function safeAIChat( required string prompt, struct params = {}, struct options = {} ) {
        var maxRetries = params.maxRetries ?: 3
        var retryDelay = params.retryDelay ?: 1000  // milliseconds

        for ( var attempt = 1; attempt <= maxRetries; attempt++ ) {
            try {
                return aiChat(
                    arguments.prompt,
                    arguments.params,
                    arguments.options
                )

            } catch ( RateLimitException e ) {
                writeLog(
                    "Rate limit hit (attempt #attempt#/#maxRetries#): #e.message#",
                    "warning"
                )

                if ( attempt < maxRetries ) {
                    // Exponential backoff
                    sleep( retryDelay * attempt )
                } else {
                    // Try fallback provider
                    return tryFallbackProvider( arguments.prompt, arguments.params )
                }

            } catch ( TimeoutException e ) {
                writeLog(
                    "Timeout (attempt #attempt#/#maxRetries#): #e.message#",
                    "error"
                )

                if ( attempt == maxRetries ) {
                    return getDefaultResponse( arguments.prompt )
                }

            } catch ( AuthenticationException e ) {
                writeLog( "Authentication failed: #e.message#", "critical" )
                notifyOps( "AI authentication failure", e )
                throw e  // Don't retry auth errors

            } catch ( any e ) {
                writeLog(
                    "AI error (attempt #attempt#/#maxRetries#): #e.message#",
                    "error"
                )

                if ( attempt == maxRetries ) {
                    notifyOps( "AI service failure", e )
                    return getDefaultResponse( arguments.prompt )
                }
            }
        }
    }

    function tryFallbackProvider( required string prompt, struct params = {} ) {
        writeLog( "Attempting fallback provider", "info" )

        var fallbackProviders = [ "claude", "gemini", "groq" ]

        for ( var provider in fallbackProviders ) {
            try {
                return aiChat(
                    arguments.prompt,
                    arguments.params,
                    { provider: provider }
                )
            } catch ( any e ) {
                writeLog( "Fallback provider #provider# failed: #e.message#", "warning" )
            }
        }

        throw "All AI providers failed"
    }

    function getDefaultResponse( required string prompt ) {
        // Return safe fallback response
        return "I apologize, but I'm experiencing technical difficulties. " &
               "Please try again in a moment or contact support if the issue persists."
    }

    function notifyOps( required string message, any error ) {
        // Send to monitoring system
        // Slack, PagerDuty, email, etc.
    }
}
```

### Circuit Breaker Pattern

Prevent cascading failures:

```javascript
class singleton {
    property name="failures" type="numeric" default="0";
    property name="lastFailure" type="date";
    property name="state" type="string" default="CLOSED";  // CLOSED, OPEN, HALF_OPEN
    property name="threshold" type="numeric" default="5";
    property name="timeout" type="numeric" default="60";  // seconds

    function call( required function operation ) {
        if ( variables.state == "OPEN" ) {
            if ( dateDiff( "s", variables.lastFailure, now() ) > variables.timeout ) {
                variables.state = "HALF_OPEN"
                writeLog( "Circuit breaker entering HALF_OPEN state" )
            } else {
                throw "Circuit breaker is OPEN - service unavailable"
            }
        }

        try {
            var result = arguments.operation()

            if ( variables.state == "HALF_OPEN" ) {
                reset()
            }

            return result

        } catch ( any e ) {
            recordFailure()
            throw e
        }
    }

    function recordFailure() {
        variables.failures++
        variables.lastFailure = now()

        if ( variables.failures >= variables.threshold ) {
            variables.state = "OPEN"
            writeLog( "Circuit breaker OPENED after #variables.failures# failures", "critical" )
            notifyOps( "AI circuit breaker opened" )
        }
    }

    function reset() {
        variables.failures = 0
        variables.state = "CLOSED"
        writeLog( "Circuit breaker CLOSED - service recovered" )
    }

    function getState() {
        return variables.state
    }
}

// Usage
circuitBreaker = getInstance( "CircuitBreaker" )

response = circuitBreaker.call( () => {
    return aiChat( "What is BoxLang?" )
} )
```

***

