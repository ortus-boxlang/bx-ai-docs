---
icon: sliders
---

# Server Configuration

Configure authentication, CORS, request limits, and other security settings for your MCP server.

## Creating a Server

```javascript
// Get or create a server instance (singleton by name)
server = MCPServer( "myApp" )

// Multiple servers for different purposes
apiServer = MCPServer( "api" )
adminServer = MCPServer( "admin" )
```

## Server Info

```javascript
server = MCPServer( "myApp" )
    .setDescription( "My Application MCP Server" )
    .setVersion( "2.0.0" )

// Get server info
info = server.getServerInfo()
// { name: "myApp", version: "2.0.0", description: "..." }
```

## Basic Authentication 🔒

Protect your server with HTTP Basic Authentication:

```javascript
server = MCPServer( "myApp" )
    .withBasicAuth( "admin", "secretPassword123" )
    .registerTool( myTool )
```

**How it works:**
- Credentials verified before any request processing
- Returns `401 Unauthorized` if invalid
- Uses standard HTTP Basic Authentication (base64-encoded `username:password`)
- Zero overhead when not configured

**Making authenticated requests:**

```bash
# Using curl with basic auth
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -u admin:secretPassword123 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'
```

**Check if enabled:**

```javascript
if ( server.hasBasicAuth() ) {
    writeOutput( "This server requires authentication" )
}
```

**Best Practices:**
- ✅ Always use HTTPS in production
- ✅ Store passwords in environment variables
- ✅ Use strong, unique passwords
- ✅ Combine with CORS settings
- ✅ Log authentication failures

**Example with environment variables:**

```javascript
MCPServer( "admin" )
    .withBasicAuth( getEnv( "MCP_ADMIN_USER" ), getEnv( "MCP_ADMIN_PASS" ) )
    .registerTool( adminTool )
```

## CORS Configuration 🌐

Control which origins can access your server:

```javascript
// Allow a single origin
server = MCPServer( "myApp" )
    .withCors( "https://example.com" )

// Allow multiple origins
server = MCPServer( "myApp" )
    .withCors( [ "https://example.com", "https://app.example.com" ] )

// Allow all origins (use with caution!)
server = MCPServer( "myApp" )
    .withCors( "*" )

// Wildcard subdomain matching
server = MCPServer( "myApp" )
    .withCors( [ "*.example.com", "https://trusted.org" ] )
```

**Wildcard Patterns:**
- `*.example.com` — Matches any subdomain
- `*` — Matches all origins
- Exact matches — Only specific origins

**Dynamic Management:**

```javascript
server = MCPServer( "myApp" )
    .addCorsOrigin( "https://example.com" )
    .addCorsOrigin( "https://app.example.com" )

origins = server.getCorsAllowedOrigins()

if ( server.isCorsAllowed( "https://example.com" ) ) {
    writeOutput( "Origin is allowed" )
}
```

**Security:**
- ✅ Avoid `*` in production
- ✅ Use HTTPS origins
- ✅ Combine with authentication
- ✅ Review periodically

## Request Body Size Limits 📏

Protect against large payloads:

```javascript
// Limit to 1MB
server = MCPServer( "myApp" )
    .withBodyLimit( 1048576 )

// Limit to 500KB
server = MCPServer( "myApp" )
    .withBodyLimit( 500 * 1024 )

// Unlimited (default: 0)
server = MCPServer( "myApp" )
    .withBodyLimit( 0 )
```

**How it works:**
- Checks body size before processing
- Returns `413 Payload Too Large` if exceeded
- Default is `0` (unlimited)
- Applies to entire JSON-RPC request

**Get current limit:**

```javascript
maxSize = server.getMaxRequestBodySize()
```

**Use Cases:**
- Public APIs — Prevent abuse
- Resource constraints — Match server limits
- Tool-specific limits — Different servers, different limits
- DoS prevention — Basic protection

**Example — Tiered Limits:**

```javascript
// Public API - strict limits
MCPServer( "public" )
    .withBodyLimit( 100 * 1024 )  // 100KB
    .registerTool( publicTool )

// Internal API - generous limits
MCPServer( "internal" )
    .withBodyLimit( 10 * 1024 * 1024 )  // 10MB
    .withBasicAuth( "admin", "secret" )
    .registerTool( adminTool )

// Data import - unlimited
MCPServer( "import" )
    .withBodyLimit( 0 )
    .registerTool( importTool )
```

## Custom API Key Validation 🔑

Implement custom authentication logic:

```javascript
// Simple validation
server = MCPServer( "myApp" )
    .withApiKeyProvider( ( apiKey, requestData ) => {
        return apiKey == "my-secret-key-12345"
    } )

// Database lookup
server = MCPServer( "myApp" )
    .withApiKeyProvider( ( apiKey, requestData ) => {
        var user = userService.findByApiKey( apiKey )
        return !isNull( user ) && user.isActive
    } )

// Complex validation with rate limiting
server = MCPServer( "myApp" )
    .withApiKeyProvider( ( apiKey, requestData ) => {
        var key = apiKeyService.validate( apiKey )
        if ( isNull( key ) ) return false

        if ( rateLimiter.isExceeded( key.userId ) ) {
            throw( "Rate limit exceeded", "RateLimitError" )
        }

        auditLog.record( key.userId, requestData.method )
        return true
    } )
```

**Provider Function Signature:**

```javascript
function apiKeyProvider(
    required string apiKey,
    required struct requestData
) {
    // Return true to allow, false to deny
    // Throw exception for errors
    return true
}
```

**API Key Extraction:**
The server automatically extracts keys from:
1. `X-API-Key` header
2. `Authorization: Bearer <token>` header

**Making requests:**

```bash
# Using X-API-Key header
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -H "X-API-Key: my-secret-key-12345" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'

# Using Bearer token
curl -X POST http://localhost/~bxai/mcp.bxm?server=myApp \
  -H "Authorization: Bearer my-secret-key-12345" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}'
```

**Multi-Tenant Example:**

```javascript
MCPServer( "api" )
    .withApiKeyProvider( ( apiKey, requestData ) => {
        var tenant = tenantService.validateApiKey( apiKey )
        if ( isNull( tenant ) ) return false

        if ( !tenant.hasFeature( requestData.method ) ) {
            throw( "Feature not available in your plan", "AccessDenied" )
        }

        request.tenantId = tenant.id
        request.tenantName = tenant.name

        usageTracker.record(
            tenantId: tenant.id,
            method: requestData.method,
            timestamp: now()
        )

        return true
    } )
    .registerTool( myTool )
```

## Combining Security Features

Use multiple layers together:

```javascript
server = MCPServer( "enterprise" )
    // CORS - restrict origins
    .withCors( [ "https://app.example.com", "*.example.com" ] )
    // Body limits - prevent abuse
    .withBodyLimit( 1048576 )  // 1MB
    // API keys - identify clients
    .withApiKeyProvider( validateApiKey )
    // Basic auth - admin access
    .withBasicAuth( "admin", "secret" )
    // Tools
    .registerTool( toolOne )
    .registerTool( toolTwo )
```

## Security Processing Order 🔐

When a request arrives, checks execute in this order:

1. **Body Size Check** — Reject oversized payloads first
2. **CORS Validation** — Check origin header
3. **Basic Authentication** — Verify credentials
4. **API Key Validation** — Call custom provider
5. **Request Processing** — Only after all checks pass

**Short-Circuit Behavior:**
- Each layer can immediately reject the request
- Failed checks return error responses
- Security headers always included in responses
- Events fired for all security failures

## Security Headers 🛡️

All HTTP responses automatically include industry-standard security headers:

| Header | Value | Purpose |
|---|---|---|
| X-Content-Type-Options | nosniff | Prevents MIME type sniffing |
| X-Frame-Options | DENY | Blocks iframe embedding |
| X-XSS-Protection | 1; mode=block | Enables XSS filtering |
| Referrer-Policy | strict-origin-when-cross-origin | Controls referrer leaking |
| Content-Security-Policy | default-src 'none' | Restricts resource loading |
| Strict-Transport-Security | max-age=31536000 | Forces HTTPS (HTTPS only) |
| Permissions-Policy | Disables geo/mic/camera | Disables sensitive APIs |

**No configuration needed** — Headers applied automatically!

## Next Steps

- 🧩 [Registering Tools](registration.md) — Add functionality
- 📋 [Observability](observability.md) — Monitor requests
- 🏗️ [Class-Based Servers](class-based-servers.md) — Organize complex servers
- ✅ [Best Practices](best-practices.md) — Production setup
