---
description: Limit how often one client can call your MCP server, by IP, API key, or your own key.
icon: gauge
---

# Rate Limiting

Limit how often one client can call an MCP server you build with `mcpServer()`. It is off by default.

```javascript
mcpServer( "myApp" )
    .withRateLimit( 100 )   // 100 requests per 60 seconds, per IP
```

A client over the limit gets `429 Too Many Requests` and has to wait. The check runs **before** authentication, so floods of failed logins are limited too. A CORS preflight is never counted.

## Configure it

```javascript
mcpServer( "myApp" ).withRateLimit()          // the defaults from the module settings
mcpServer( "myApp" ).withRateLimit( 100 )     // 100 per default window
mcpServer( "myApp" ).withRateLimit( {
    limit    : 100,
    window   : 60,           // seconds
    key      : "ip",         // "ip" | "apiKey" | "global" | lambda
    store    : "cache",      // "cache" | "memory" | class path | IRateLimitStore instance
    cacheName: "mcpLimits",  // which cache, "default" if empty
    failOpen : true          // store fails: allow (true) or answer 503 (false)
} )
mcpServer( "myApp" ).withRateLimit( false )   // no limit for this server
```

An explicit struct overrides the [module settings](#module-settings). A bad value throws `InvalidArgument`.

## Who is limited: `key`

| Value | Meaning |
|---|---|
| `"ip"` (default) | The client IP from `getClientIP()`. Set [trusted proxies](server-configuration.md#trusted-proxies) in production. |
| `"apiKey"` | The key from `X-API-Key` or a Bearer token, **only when your API key provider accepts it**. A missing or rejected key falls back to the client IP, so fake keys cannot dodge the limit or fill the cache. Needs `withApiKeyProvider()` first. |
| `"global"` | One shared bucket for every client of this server |
| a lambda | `( requestData ) => string`. An empty result means this request is not limited, which is how you exempt clients. |

```javascript
// One limit per tenant, internal callers exempt
server.withRateLimit( {
    limit: 1000,
    key  : ( requestData ) => {
        if ( requestData.headers[ "X-Internal" ] ?: false ) return ""
        return requestData.headers[ "X-Tenant" ] ?: ""
    }
} )
```

With the `apiKey` key, the provider is asked once per request: the answer is reused by the authentication step.

stdio has no IP. With the `ip` key every stdio request shares one bucket, so use `global` or a lambda there.

## How it counts

One sliding window counter. Time is cut in windows, each client has a counter per window, and the count that matters is `previous * (1 - elapsed) + current`. There is no burst at the window edge, and the `Retry-After` value is exact. A refused request is not counted.

## The response

| Case | Result |
|---|---|
| Allowed | The normal response plus `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset` (epoch seconds) |
| Blocked | HTTP `429`, JSON-RPC error `-32002`, the same headers plus `Retry-After` (seconds), and the request's `X-Request-ID` and CORS headers |
| Store fails | `failOpen: true` (default) lets the request through and logs an error. `false` answers `503`. |

## Where the counters live

By default in a BoxLang cache. Use a **dedicated cache** (`cacheName`), so counters are not evicted early by other entries: an evicted counter starts again at 0.

| `store` | Use |
|---|---|
| `"cache"` | The default. Counters are changed under a lock, so counts are exact inside one JVM. Across several nodes, counts are approximate. |
| `"memory"` | In this JVM only, lost on restart. Single node or tests. |
| class path or instance | Your own store, for Redis or a database |

A custom store implements `IRateLimitStore`:

```javascript
class implements="bxModules.bxai.models.mcp.ratelimit.IRateLimitStore" {

    // Add `amount` to the counter and return the new value. The counter expires after ttlSeconds.
    numeric function increment( required string key, required numeric ttlSeconds, numeric amount = 1 ) { ... }

    // The current value, 0 when there is none
    numeric function get( required string key ) { ... }

    void function remove( required string key ) { ... }

}
```

```javascript
server.withRateLimit( { limit: 100, store: "models.RedisRateLimitStore" } )
// or an instance
server.withRateLimit( { limit: 100, store: new models.RedisRateLimitStore( redis ) } )
```

A class path is created with `cacheName` as its only argument. Use an instance when you need more.

Counter keys look like `bxai:mcp:ratelimit:<server>:<hash>:<window number>`. The client key is hashed, and counters expire after two windows.

## Logs

Blocked requests are **never logged one by one**, so a flood cannot flood your logs.

- A client's first block in a window can write one warning on the `mcp` logger, with the client (an IP is logged plain, API keys and custom keys as a short hash), the limit and the retry time.
- Each server writes at most **10** of those warnings per minute. This is not a setting.
- When lines were suppressed, one summary info line follows: requests blocked, clients, lines suppressed.
- Store and key lambda errors are logged at most once per minute.
- Allowed requests are never logged.

## Stats

`getStats()` includes `security.rateLimitFailures` and a `rateLimit` section. The totals are exact, whatever the log budget did.

```javascript
stats = server.getStats()
stats.security.rateLimitFailures
stats.rateLimit
// { keyType, limit, window, allowed, blocked, storeErrors,
//   topBlocked : [ { client, blocked, lastBlockedAt } ] }   // at most 20, so memory stays flat
```

`resetStats()` clears them.

## Event

`onMCPRateLimitHit` fires on a client's first block in a window and shares the log budget of 10 per minute. See [events](../../advanced/events.md#71-onmcpratelimithit).

## Look at or reset a client

```javascript
limiter = server.getRateLimiter()
limiter.status( "ip:203.0.113.5" )   // { limit, remaining, resetAt }
limiter.reset( "ip:203.0.113.5" )    // give the client its requests back
```

Bucket keys are prefixed with their type: `ip:`, `apiKey:`, `custom:`, and `global`.

## Module settings

Defaults for every MCP server, in `boxlang.json`:

```json
"mcp": {
    "server": {
        "trustedProxies": [],
        "rateLimit": {
            "enabled"  : false,
            "limit"    : 60,
            "window"   : 60,
            "key"      : "ip",
            "store"    : "cache",
            "cacheName": "default",
            "failOpen" : true
        }
    }
}
```

With `enabled: true` every MCP server is limited with no code change. With `key: "apiKey"` a server is limited once it has an API key provider. `withRateLimit( false )` opts one server out.

## Next Steps

- 🔧 [Server Configuration](server-configuration.md) for trusted proxies, authentication and the other security checks
- 📊 [Observability](observability.md) for stats and events
- ⭐ [Best Practices](best-practices.md)
