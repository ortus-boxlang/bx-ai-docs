---
description: >-
  Environment-specific security configuration, security headers, and
  network-level hardening for AI endpoints.
icon: network-wired
---

# Network & Configuration Hardening

## Secure Configuration

### Environment-Specific Settings

```javascript
// config/security.bx
class {
    function getSecurityConfig() {
        var env = getSystemSetting( "ENVIRONMENT", "production" )

        var configs = {
            "development": {
                enableAuditLogging: false,
                requireEncryption: false,
                enablePIIDetection: false,
                allowLocalProviders: true,
                maxInputLength: 100000,
                rateLimitPerMinute: 100
            },
            "staging": {
                enableAuditLogging: true,
                requireEncryption: true,
                enablePIIDetection: true,
                allowLocalProviders: true,
                maxInputLength: 50000,
                rateLimitPerMinute: 50
            },
            "production": {
                enableAuditLogging: true,
                requireEncryption: true,
                enablePIIDetection: true,
                allowLocalProviders: false,
                maxInputLength: 10000,
                rateLimitPerMinute: 20,
                enableCircuitBreaker: true,
                enableRateLimiting: true
            }
        }

        return configs[ env ]
    }
}
```

### Security Headers

```javascript
// Application.bx
class {
    function onRequestStart() {
        // Security headers
        bx:header name="X-Content-Type-Options", value="nosniff";
        bx:header name="X-Frame-Options", value="DENY";
        bx:header name="X-XSS-Protection", value="1; mode=block";
        bx:header name="Strict-Transport-Security", value="max-age=31536000; includeSubDomains";
        bx:header name="Content-Security-Policy", value="default-src 'self'";
        bx:header name="Referrer-Policy", value="no-referrer";
    }
}
```

***

## Network Security

### API Gateway

**Route all AI requests through secure gateway**:

```javascript
class {
    function proxyAIRequest(
        required string prompt,
        struct params = {}
    ) {
        // Authenticate
        if ( !isAuthenticated() ) {
            throw "Unauthorized"
        }

        // Rate limit
        if ( !checkRateLimit( session.user.id ) ) {
            throw "Rate limit exceeded"
        }

        // Validate input
        validateInput( arguments.prompt )

        // Log request
        logAIInteraction( "proxy_request", session.user.id, arguments.prompt )

        // Call AI (with API key from secure storage)
        var apiKey = getAPIKeyFromVault( arguments.params.provider ?: "openai" )
        var response = aiChat(
            arguments.prompt,
            arguments.params.append({ apiKey: apiKey })
        )

        // Validate output
        validateOutput( response )

        // Log response
        logAIInteraction( "proxy_response", session.user.id, arguments.prompt, response )

        return response
    }
}
```

### TLS/SSL

**Require HTTPS for all AI endpoints**:

```javascript
// Application.bx
function onRequestStart() {
    // Force HTTPS in production
    if ( getSystemSetting( "ENVIRONMENT" ) == "production" &&
         !cgi.https
) {
        bx:location "https://#cgi.server_name##cgi.script_name#";
    }
}
```
