---
description: Getting started with web search - basic usage, async patterns, agent integration
icon: rocket
---

# Getting Started with Web Search

Learn how to integrate web search into your BoxLang AI applications with practical examples and patterns.

## Basic Usage

### Simple Search

The simplest way to search:

```javascript
// Search with default HTTP provider
var results = aiWebSearch( "BoxLang programming language" )

// Results is an array of search result structs
results.each( result => {
    println( "Title: #result.title#" )
    println( "URL: #result.url#" )
    println( "Snippet: #result.snippet#" )
    println( "---" )
} )
```

### Selecting a Provider

```javascript
// Use HTTP (default) - no API key needed
var results = aiWebSearch( "AI news", { provider: "http" } )

// Use Brave Search
var results = aiWebSearch(
    "AI news",
    { provider: "brave", maxResults: 10 }
)

// Use Google Custom Search
var results = aiWebSearch(
    "BoxLang",
    { provider: "google", maxResults: 5 }
)

// Use Tavily (AI-optimized)
var results = aiWebSearch(
    "latest AI models",
    { provider: "tavily", maxResults: 15 }
)

// Use Exa (neural/semantic)
var results = aiWebSearch(
    "semantic search techniques",
    { provider: "exa", maxResults: 5 }
)
```

### Limiting Results

```javascript
// Get just top 3 results
var topResults = aiWebSearch(
    "BoxLang",
    { maxResults: 3 }
)

// Often enough for agents to make decisions
for ( result in topResults ) {
    println( result.title )
}
```

## Async/Non-Blocking Search

For better performance when not blocking on results:

```javascript
// Start async search
var future = aiWebSearchAsync( "BoxLang updates" )

// Do other work while searching...
var config = loadConfiguration()
var cache = initCache()

// Get results when ready
var results = future.get()  // Blocks until ready

// Or use callback pattern
aiWebSearchAsync( "latest news" )
    .then( results => {
        println( "Got #results.len()# results" )
        return results
    } )
    .catch( error => {
        println( "Search failed: #error#" )
    } )
```

## Working with Results

### Filtering Results

```javascript
function filterByDomain( required array results, required string domain ) {
    return results.filter( r => r.domain.contains( domain ) )
}

var results = aiWebSearch( "BoxLang" )
var officialResults = filterByDomain( results, "ortus" )
```

### Extracting Data

```javascript
// Get just URLs
var urls = results.map( r => r.url )

// Get just snippets
var snippets = results.map( r => r.snippet )

// Custom extraction
var summaries = results.map( r => {
    return {
        title: r.title,
        url: r.url,
        domain: r.domain,
        score: r.score
    }
} )
```

### Sorting Results

```javascript
// Sort by relevance score (default order)
var sorted = results.sort( (a, b) => b.score - a.score )

// Sort by date (newest first)
var sorted = results.sort( (a, b) => {
    dateCompare = dateCompare( b.publishedDate, a.publishedDate )
    return dateCompare
} )

// Sort by domain
var sorted = results.sort( (a, b) => a.domain.compareNoCase( b.domain ) )
```

## Agent Integration

### Basic Agent with Web Search

```javascript
// Create an agent with web search capability
var agent = aiAgent(
    name: "WebAssistant",
    description: "An assistant with web search abilities",
    instructions: "Use web search to find current information and answer questions accurately.",
    tools: [ "webSearch@bxai" ]  // Add auto-registered web search tool
)

// Agent uses web search automatically when needed
var response = agent.run( "What are the latest BoxLang releases?" )
println( response )
```

### Multi-Tool Agent

```javascript
// Combine web search with other tools
var searchTool = "webSearch@bxai"

var databaseLookupTool = aiTool(
    "lookup_user",
    "Look up a user in the database",
    ( userId ) => {
        var result = queryExecute(
            "SELECT * FROM users WHERE id = :id",
            { id: userId }
        )
        return result.recordCount > 0 ? result : "User not found"
    }
)

var agent = aiAgent(
    name: "DataAgent",
    tools: [ searchTool, databaseLookupTool ]
)

// Agent can now search web AND query database
var response = agent.run(
    "Find information about BoxLang online and also look up user ID 123 in our system"
)
```

### Research Task Agent

```javascript
var researchAgent = aiAgent(
    name: "Researcher",
    description: "Research assistant for fact-checking and information gathering",
    instructions:
        "Your job is to research topics thoroughly. " &
        "Use web search to find reliable sources. " &
        "Always cite your sources from the search results.",
    tools: [ "webSearch@bxai" ]
)

// Research a topic
var research = researchAgent.run(
    "Research the current state of BoxLang and summarize what you find. Include URLs of your sources."
)

println( research )
```

## With Memory

### Combining Web Search and Conversation Memory

```javascript
// Create memory for conversation history
var memory = aiMemory( memory: "window", config: {
    maxMessages: 20
} )

// Create agent with both web search and memory
var agent = aiAgent(
    name: "ContextualAssistant",
    tools: [ "webSearch@bxai" ],
    memories: [ memory ]
)

// First question
var response1 = agent.run( "Search for information about BoxLang" )
println( response1 )

// Follow-up - agent remembers previous context
var response2 = agent.run( "What was mentioned about cloud deployment?" )
// Agent can reference previous web search results from memory
```

### Web Search + Vector Memory (RAG)

```javascript
// Create vector memory for storing web search results
var vectorMemory = aiMemory( memory: "chroma", config: {
    collection: "research",
    embeddingProvider: "openai"
} )

// Search and store results
var results = aiWebSearch( "BoxLang framework" )

// Add results to vector memory
for ( result in results ) {
    vectorMemory.add(
        result.snippet,
        {
            source: "web_search",
            title: result.title,
            url: result.url,
            domain: result.domain
        }
    )
}

// Create agent with vector memory
var agent = aiAgent(
    name: "KnowledgeAssistant",
    tools: [ "webSearch@bxai" ],
    memories: [ vectorMemory ]
)

// Agent can now reference both current web search and previous research
var response = agent.run(
    "Summarize what you know about BoxLang architecture and framework features"
)
```

## Error Handling

### Catching Search Errors

```javascript
function safeWebSearch( required string query ) {
    try {
        return aiWebSearch( arguments.query, {
            provider: "brave",
            timeout: 10
        } )
    } catch ( any error ) {
        writeLog( "Web search failed: #error.message#", "error" )

        // Fallback to HTTP provider
        return aiWebSearch( arguments.query, { provider: "http" } )
    }
}

var results = safeWebSearch( "BoxLang" )
```

### Rate Limiting

```javascript
function limitedWebSearch(
    required string userId,
    required string query
) {
    // Check if user has exceeded rate limit
    var searchCount = queryExecute(
        "SELECT COUNT(*) as cnt FROM web_searches
         WHERE user_id = :userId AND created_at > DATE_SUB(NOW(), INTERVAL 1 HOUR)",
        { userId: arguments.userId }
    )

    if ( searchCount.cnt >= 50 ) {
        throw "You have exceeded your search limit (50 per hour)"
    }

    // Execute search and log
    var results = aiWebSearch( arguments.query )

    queryExecute(
        "INSERT INTO web_searches (user_id, query, result_count, created_at)
         VALUES (:userId, :query, :resultCount, :createdAt)",
        {
            userId: arguments.userId,
            query: arguments.query,
            resultCount: results.len(),
            createdAt: now()
        }
    )

    return results
}
```

## Performance Tips

### Caching Results

```javascript
var cache = createObject( "java", "java.util.concurrent.ConcurrentHashMap" )

function cachedWebSearch( required string query, required string provider = "brave" ) {
    var cacheKey = provider & ":" & query

    // Check cache
    if ( cache.containsKey( cacheKey ) ) {
        var cached = cache.get( cacheKey )
        if ( dateAdd( "h", 1, cached.timestamp ) > now() ) {
            writeLog( "Cache hit: #query#" )
            return cached.results
        }
    }

    // Execute search
    var results = aiWebSearch( arguments.query, { provider: arguments.provider } )

    // Cache result
    cache.put( cacheKey, {
        results: results,
        timestamp: now()
    } )

    return results
}
```

### Parallel Searches

```javascript
// Search multiple providers in parallel for comparison
function parallelWebSearch( required string query ) {
    var future1 = aiWebSearchAsync( query, { provider: "brave" } )
    var future2 = aiWebSearchAsync( query, { provider: "tavily" } )

    // Results fetch in parallel
    var braveResults = future1.get()
    var tavilyResults = future2.get()

    return {
        brave: braveResults,
        tavily: tavilyResults
    }
}

var results = parallelWebSearch( "latest AI news" )
```

## Next Steps

- 📖 **[Providers Reference](providers.md)** - Detailed documentation for each provider
- 🤖 **[Agent Integration](agent-tools.md)** - Advanced agent patterns
- 🔍 **[API Reference](../../advanced/reference/built-in-functions/aiwebsearch.md)** - Complete BIF documentation
- 🛡️ **[Security](../../deployment/security.md#-web-search-specific-security)** - Security best practices
