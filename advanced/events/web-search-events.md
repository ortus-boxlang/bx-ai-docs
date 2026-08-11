---
description: Events fired around AI-driven web search requests, responses, and errors.
icon: globe
---

# Web Search Events

### beforeAIWebSearch

Fired before a web search provider executes a query.

**When**: At the start of `BaseSearch.search()`, before the provider runs

| Argument | Type | Description |
| --- | --- | --- |
| `provider` | `ISearchProvider` | The search provider instance |
| `query` | `String` | The search query |
| `options` | `Struct` | Search options (maxResults, etc.) |

### afterAIWebSearch

Fired after a web search returns results successfully.

| Argument | Type | Description |
| --- | --- | --- |
| `provider` | `ISearchProvider` | The search provider instance |
| `query` | `String` | The search query |
| `options` | `Struct` | Search options |
| `results` | `Array` | Normalized result array |
| `cached` | `Boolean` | Whether the results came from cache |

```javascript
BoxRegisterInterceptor( "afterAIWebSearch", function( event ) {
    println( "Search '#event.query#' returned #event.results.len()# results" )
})
```

### onAIWebSearchRequest

Fired immediately before the provider's outbound HTTP request.

| Argument | Type | Description |
| --- | --- | --- |
| `provider` | `ISearchProvider` | The search provider instance |
| `url` | `String` | The request URL |
| `method` | `String` | HTTP method |
| `headers` | `Struct` | Request headers |

### onAIWebSearchResponse

Fired after the provider's HTTP response is received.

| Argument | Type | Description |
| --- | --- | --- |
| `provider` | `ISearchProvider` | The search provider instance |
| `url` | `String` | The request URL |
| `statusCode` | `Numeric` | HTTP status code |
| `response` | `Struct` | The raw HTTP response |

### onAIWebSearchError

Fired when a web search throws.

| Argument | Type | Description |
| --- | --- | --- |
| `provider` | `ISearchProvider` | The search provider instance |
| `query` | `String` | The search query |
| `options` | `Struct` | Search options |
| `error` | `Exception` | The thrown exception |
