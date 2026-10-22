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
mcpServer( "myApp" ).withRateLimit( false )   // remove the limit from this server
```

An explicit struct overrides the [module settings](#module-settings). A bad value throws `InvalidArgument`.

## Several limits

Give a server a limit per minute and a limit per hour, or any other windows:

```javascript
mcpServer( "myApp" ).withRateLimit( {
    limits: [
        { limit: 60,   window: 60 },     // 60 per minute
        { limit: 1000, window: 3600 }    // 1000 per hour
    ]
} )
```

A request has to pass **all** the limits. When one refuses, the other limits get the request back, so a refused call never uses up an allowance. `Retry-After` is the longest of the waits, and the `X-RateLimit-*` headers describe the limit the client is closest to. Each limit needs its own window. `limits` is used instead of `limit` and `window`.

## Turn it on and off

```javascript
server.disableRateLimit()      // stop checking, keep the setup, counters and history
server.enableRateLimit()       // check again (uses the module defaults if no limit was set)
server.isRateLimitEnabled()
server.hasRateLimit()          // a limit is configured, on or off
```

While it is off, requests are not counted or limited and no history is added. `withRateLimit( false )` removes the limit for good. BxLens shows the same switch to an admin.

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
// { enabled, keyType, limit, window, limits : [ { limit, window } ],
//   allowed, blocked, storeErrors, retentionDays,
//   topBlocked : [ { client, type, requests, blocked, peakPerMinute, pctOfLimit, lastBlockedAt } ] }
```

`allowed`, `blocked` and `storeErrors` count since the server started. `topBlocked` is today's top 20, so it recycles every UTC day. `resetStats()` clears the numbers, and `resetRateLimitStats()` also clears the history.

## History and reports

The server remembers what each client did, per client and per UTC day: requests, blocked requests, the busiest minute, and the last block. It keeps this in the BoxLang cache for `retention` days, so it survives a restart and nodes that share the cache see each other's numbers.

```javascript
server.withRateLimit( { limit: 60, cacheName: "mcpLimits", retention: 30 } )

report = server.getRateLimitReport( { days: 7, top: 20 } )
// { since, nodes, retentionDays,
//   days         : [ { day, requests, blocked } ],
//   topConsumers : [ { client, type, requests, blocked, peakPerMinute, pctOfLimit, lastBlockedAt } ] }
```

- `days` counts today as the first day. `top` is the number of clients, the busiest by requests first.
- `pctOfLimit` is the busiest minute of the client against the limit of the shortest window, scaled to a minute. Over 100 means the client hit the limit.
- IPs show plain. API keys and custom keys show as a short hash, like in the logs.
- `retention` is in days: the module setting `retentionDays` (default 7, 0 for none, at most 365) unless the server sets its own. With 0 only today is kept, in memory.

**How it stays small and fast.** A request only changes numbers in memory. About every `historyFlushSeconds` (default 30, at least 5) the next request writes the day to the cache, with a time to live of the retention, so old days disappear by themselves. A day keeps at most `historyMaxClients` clients (default 500, from 50 to 5,000). When it is full, the smallest clients are folded into one `(other)` row: the totals stay exact and only the detail of the smallest is dropped. There is one document per server, node and day, so nodes never overwrite each other. A restarted node continues its day.

If the process stops, up to `historyFlushSeconds` of history is lost. The counters that enforce the limit are written on every request and are not affected.

## Reset

```javascript
server.resetRateLimit()         // every client has its requests back
server.resetRateLimitStats()    // clear the stats and every day of history
```

`resetRateLimit()` deletes the counters from the store when the store implements the optional `IClearableRateLimitStore` (`clear( prefix )`). The cache and memory stores do. It returns `false` when it cannot, and then the counters expire by themselves within two windows. `limiter.reset( key )` still resets one client.

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
            "failOpen" : true,
            "retentionDays"       : 7,
            "historyFlushSeconds" : 30,
            "historyMaxClients"   : 500
        }
    }
}
```

`retentionDays`, `historyFlushSeconds` and `historyMaxClients` are the defaults for [history](#history-and-reports), and a server can override them with `retention`, `historyFlushSeconds` and `historyMaxClients`. With `enabled: true` every MCP server is limited with no code change. With `key: "apiKey"` a server is limited once it has an API key provider. `withRateLimit( false )` opts one server out.

## Next Steps

- 🔧 [Server Configuration](server-configuration.md) for trusted proxies, authentication and the other security checks
- 📊 [Observability](observability.md) for stats and events
- ⭐ [Best Practices](best-practices.md)
