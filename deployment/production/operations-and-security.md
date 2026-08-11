---
description: >-
  Security hardening and operational procedures for running BoxLang AI in
  production.
icon: shield-halved
---

# Operations & Security

## 🔐 Security Hardening

### API Key Rotation

```javascript
class singleton {
    property name="apiKeys" type="struct";
    property name="currentKeyIndex" type="struct";

    function init() {
        variables.apiKeys = {}
        variables.currentKeyIndex = {}
        loadKeys()
        return this
    }

    function loadKeys() {
        // Load multiple keys per provider
        variables.apiKeys = {
            "openai": [
                getSystemSetting( "OPENAI_API_KEY_1" ),
                getSystemSetting( "OPENAI_API_KEY_2" )
            ],
            "claude": [
                getSystemSetting( "CLAUDE_API_KEY_1" ),
                getSystemSetting( "CLAUDE_API_KEY_2" )
            ]
        }

        // Initialize index
        for ( var provider in variables.apiKeys ) {
            variables.currentKeyIndex[ provider ] = 1
        }
    }

    function getAPIKey( required string provider ) {
        var keys = variables.apiKeys[ arguments.provider ]
        var index = variables.currentKeyIndex[ arguments.provider ]

        return keys[ index ]
    }

    function rotateKey( required string provider ) {
        lock name="keyrotation_#arguments.provider#" type="exclusive" timeout="5" {
            var keys = variables.apiKeys[ arguments.provider ]
            variables.currentKeyIndex[ arguments.provider ]++

            if ( variables.currentKeyIndex[ arguments.provider ] > arrayLen( keys ) ) {
                variables.currentKeyIndex[ arguments.provider ] = 1
            }

            writeLog( "Rotated API key for provider: #arguments.provider#" )
        }
    }
}
```

### Request Validation

```javascript
function validateAIRequest( required struct request ) {
    // Check for injection attempts
    if ( request.prompt.findNoCase( "ignore previous" ) ||
         request.prompt.findNoCase( "disregard instructions" ) ) {
        writeLog(
            "Potential prompt injection detected: #left( request.prompt, 100 )#",
            "security"
        )
        throw "Invalid request"
    }

    // Rate limiting per user
    if ( !checkRateLimit( session.userId ) ) {
        throw "Rate limit exceeded"
    }

    // Input size limits
    if ( len( request.prompt ) > 50000 ) {
        throw "Prompt exceeds maximum length"
    }

    return true
}
```

**More security details**: [Security Guide](../security/README.md)

***


## 🔧 Operational Procedures

### Deployment Steps

1.  **Pre-deployment**:

    ```bash
    # Run tests
    box task run test

    # Validate configuration
    box task run validate:config

    # Build artifacts
    box task run build
    ```
2.  **Deploy**:

    ```bash
    # Blue-green deployment
    kubectl apply -f deployment-blue.yaml
    kubectl rollout status deployment/boxlang-ai-blue

    # Switch traffic
    kubectl patch service boxlang-ai -p '{"spec":{"selector":{"version":"blue"}}}'

    # Monitor
    kubectl logs -f deployment/boxlang-ai-blue
    ```
3.  **Verify**:

    ```bash
    # Health check
    curl https://api.example.com/health

    # Smoke tests
    box task run test:smoke
    ```
4.  **Rollback** (if needed):

    ```bash
    # Switch back to green
    kubectl patch service boxlang-ai -p '{"spec":{"selector":{"version":"green"}}}'
    ```

### Monitoring Alerts

Configure alerts for:

* ❗ **Error rate > 5%** - High error threshold
* ❗ **Response time > 10s** - Performance degradation
* ❗ **Cost > daily budget** - Budget exceeded
* ❗ **Circuit breaker open** - Service unavailable
* ❗ **Provider failover** - Backup provider activated
* ⚠️ **Memory usage > 80%** - Resource warning
* ⚠️ **Token usage spike** - Unusual activity

### Incident Response

**AI service outage**:

1. Check provider status pages
2. Attempt provider failover
3. Enable caching of recent responses
4. Activate maintenance mode if necessary
5. Notify users of degraded service
6. Document incident and resolution

***

