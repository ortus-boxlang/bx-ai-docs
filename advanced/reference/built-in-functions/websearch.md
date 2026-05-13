# webSearch

Search the web using the configured or selected provider and return normalized results.

## Syntax

```javascript
webSearch( query, options = {} )
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Search query text |
| `options` | struct | No | Search options including provider, limits, filters, and logging |

## Common Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | module default or `http` | Provider: `http`, `brave`, `google`, `tavily`, `exa` |
| `maxResults` | numeric | `5` | Maximum number of results |
| `timeout` | numeric | `30` | HTTP timeout in seconds |
| `logRequest` | boolean | `false` | Log provider request details |
| `logResponse` | boolean | `false` | Log provider response details |

## Provider-Specific Options

### Brave

| Option | Description |
|---|---|
| `country` | Country filter |
| `language` | Language filter |
| `safeSearch` | Safe search mode |

### Google

| Option | Description |
|---|---|
| `gl` | Country target |
| `hl` | Interface language |
| `safe` | Safe search mode |

### Tavily

| Option | Description |
|---|---|
| `topic` | Topic mode |
| `includeAnswer` | Include summary answer |
| `includeRawContent` | Include raw page content |
| `days` | Recency in days |

### Exa

| Option | Description |
|---|---|
| `type` | `keyword`, `neural`, or `magic` |
| `country` | Country filter |
| `language` | Language filter |

## Returns

Array of structs with normalized fields:

```javascript
[
    {
        title: "...",
        url: "...",
        snippet: "...",
        publishedDate: "...",
        domain: "...",
        score: 0.95,
        thumbnail: "...",
        language: "en"
    }
]
```

## Events Fired

| Event | When |
|---|---|
| `beforeAIWebSearch` | Before provider search execution |
| `onAIWebSearchRequest` | Right before HTTP request dispatch |
| `onAIWebSearchResponse` | After provider HTTP response |
| `afterAIWebSearch` | After successful search completion |
| `onAIWebSearchError` | On provider/search error |

## Examples

### Default provider

```javascript
results = webSearch( "BoxLang AI" )
```

### Brave search with filters

```javascript
results = webSearch(
    "latest BoxLang news",
    {
        provider: "brave",
        maxResults: 10,
        country: "us",
        language: "en",
        safeSearch: "moderate"
    }
)
```

### Exa neural search

```javascript
results = webSearch(
    "agent memory retrieval patterns",
    {
        provider: "exa",
        type: "neural",
        maxResults: 8
    }
)
```

### Agent usage

```javascript
agent = aiAgent(
    name: "ResearchAgent",
    tools: [ "webSearch@bxai" ]
)

response = agent.run( "Find the latest BoxLang MCP updates" )
```
