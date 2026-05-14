---
description: Complete provider reference for Web Search in BoxLang AI
icon: globe
---

# Web Search Providers

BoxLang AI supports multiple web search providers behind a single interface. All providers normalize output to the same 8-field result format.

## Result Format

Every provider returns this structure:

```javascript
{
    title: "Result Title",
    url: "https://example.com/page",
    snippet: "Result summary text",
    publishedDate: "2026-05-14",
    domain: "example.com",
    score: 0.92,
    thumbnail: "https://example.com/image.jpg",
    language: "en"
}
```

## Provider Comparison

| Provider | Key Required | Best For | Notes |
|---|---|---|---|
| `http` | No | URL fetching and local testing | No search index, fetches/parses target URL |
| `brave` | `BRAVE_API_KEY` | Privacy-focused web search | Free tier available |
| `google` | `GOOGLE_API_KEY` + `GOOGLE_SEARCH_ENGINE_ID` | Broad web coverage | Requires Custom Search setup |
| `tavily` | `TAVILY_API_KEY` | AI-focused retrieval | Good for agent workflows |
| `exa` | `EXA_API_KEY` | Neural and semantic search | Supports `keyword`, `neural`, `magic` |

## HTTP Provider (`http`)

Use the HTTP provider when you want to fetch and parse a specific URL without external search APIs.

```javascript
results = aiWebSearch(
    "https://boxlang.io",
    {
        provider: "http",
        maxResults: 5
    }
)
```

### Options

| Option | Type | Default | Description |
|---|---|---|---|
| `maxResults` | numeric | `5` | Maximum results returned |
| `timeout` | numeric | `30` | HTTP timeout in seconds |
| `logRequest` | boolean | `false` | Log request details |
| `logResponse` | boolean | `false` | Log response details |

## Brave Provider (`brave`)

```javascript
results = aiWebSearch(
    "latest BoxLang release",
    {
        provider: "brave",
        maxResults: 10,
        country: "us",
        language: "en",
        safeSearch: "moderate"
    }
)
```

### Authentication

```bash
export BRAVE_API_KEY="your-key"
```

### Options

| Option | Type | Default | Description |
|---|---|---|---|
| `country` | string | `"us"` | Country filter |
| `language` | string | `"en"` | Language filter |
| `safeSearch` | string | `"moderate"` | Safe search mode |
| `maxResults` | numeric | `5` | Maximum results |
| `timeout` | numeric | `30` | Request timeout |

## Google Provider (`google`)

```javascript
results = aiWebSearch(
    "BoxLang AI module",
    {
        provider: "google",
        maxResults: 10,
        gl: "us",
        hl: "en",
        safe: "active"
    }
)
```

### Authentication

```bash
export GOOGLE_API_KEY="your-google-key"
export GOOGLE_SEARCH_ENGINE_ID="your-search-engine-id"
```

### Options

| Option | Type | Default | Description |
|---|---|---|---|
| `gl` | string | `"us"` | Country target |
| `hl` | string | `"en"` | Interface language |
| `safe` | string | `"off"` | Safe search filter |
| `maxResults` | numeric | `5` | Maximum results |
| `timeout` | numeric | `30` | Request timeout |

## Tavily Provider (`tavily`)

```javascript
results = aiWebSearch(
    "what changed in BoxLang AI 3.2.0",
    {
        provider: "tavily",
        maxResults: 8,
        topic: "general",
        includeAnswer: true,
        includeRawContent: false,
        days: 30
    }
)
```

### Authentication

```bash
export TAVILY_API_KEY="your-tavily-key"
```

### Options

| Option | Type | Default | Description |
|---|---|---|---|
| `topic` | string | `"general"` | Search topic mode |
| `includeAnswer` | boolean | `false` | Include provider summary answer |
| `includeRawContent` | boolean | `false` | Include raw content from source |
| `days` | numeric | `0` | Recency filter in days |
| `maxResults` | numeric | `5` | Maximum results |
| `timeout` | numeric | `30` | Request timeout |

## Exa Provider (`exa`)

```javascript
results = aiWebSearch(
    "vector memory techniques",
    {
        provider: "exa",
        maxResults: 10,
        type: "neural",
        country: "us",
        language: "en"
    }
)
```

### Authentication

```bash
export EXA_API_KEY="your-exa-key"
```

### Options

| Option | Type | Default | Description |
|---|---|---|---|
| `type` | string | `"magic"` | Search type: `keyword`, `neural`, `magic` |
| `country` | string | `"us"` | Country filter |
| `language` | string | `"en"` | Language filter |
| `maxResults` | numeric | `5` | Maximum results |
| `timeout` | numeric | `30` | Request timeout |

## Provider Selection Strategy

1. Start with `http` for local testing and deterministic URL ingestion.
2. Use `brave` or `google` for broad web search.
3. Use `tavily` when agents need retrieval-focused answers.
4. Use `exa` for semantic discovery and neural relevance.

## Related

- [Overview](README.md)
- [Getting Started](getting-started.md)
- [Agent Integration](agent-tools.md)
- [Security Guide](../../deployment/security.md)
