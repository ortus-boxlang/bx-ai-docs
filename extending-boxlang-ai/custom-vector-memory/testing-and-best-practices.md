---
description: >-
  Unit testing strategies and best practices for building production-ready
  custom vector memory implementations.
icon: vial
---

# Testing & Best Practices

## Testing Your Custom Vector Memory

### Unit Test Example

```js
class extends="testbox.system.BaseSpec" {

    function run() {
        describe( "ElasticsearchVectorMemory", () => {

            beforeEach( () => {
                variables.memory = new ElasticsearchVectorMemory().configure({
                    collection: "test_memory",
                    esHost: "localhost",
                    esPort: 9200,
                    embeddingProvider: "openai",
                    embeddingModel: "text-embedding-3-small",
                    dimensions: 1536,
                    metric: "cosine"
                });
                variables.memory.key( "test-session-#createUUID()#" );
            });

            afterEach( () => {
                variables.memory.clear();
            });

            it( "can add and retrieve messages", () => {
                // Add messages
                memory.add( "BoxLang is a modern JVM language" );
                memory.add( "It has excellent Java interop" );
                memory.add( "The weather is sunny today" );

                expect( memory.count() ).toBe( 3 );

                // Search for relevant content
                var results = memory.getRelevant( "Tell me about BoxLang", 2 );

                expect( results ).toBeArray();
                expect( results.len() ).toBeGTE( 1 );
                expect( results[ 1 ].content ).toInclude( "BoxLang" );
            });

            it( "returns results sorted by relevance", () => {
                memory.add( "Python is a programming language" );
                memory.add( "BoxLang is built on the JVM" );
                memory.add( "Java runs on the JVM" );

                var results = memory.getRelevant( "JVM languages", 3 );

                // First result should be most relevant
                expect( results[ 1 ].score ).toBeGT( results[ 2 ].score );
            });

            it( "can clear all messages", () => {
                memory.add( "Test message 1" );
                memory.add( "Test message 2" );

                expect( memory.count() ).toBe( 2 );

                memory.clear();

                expect( memory.count() ).toBe( 0 );
            });

            it( "isolates messages by key", () => {
                memory.key( "session-1" );
                memory.add( "Message in session 1" );

                memory.key( "session-2" );
                memory.add( "Message in session 2" );

                memory.key( "session-1" );
                var session1Messages = memory.getAll();

                expect( session1Messages.len() ).toBe( 1 );
                expect( session1Messages[ 1 ].content ).toBe( "Message in session 1" );
            });

        });
    }
}
```


## Best Practices

### 1. Always Call `super.configure()`

```js
function configure( required struct config ) {
    // CRITICAL: Call parent configuration first
    super.configure( arguments.config );

    // Then add your custom configuration
    variables.myCustomSetting = arguments.config.myCustomSetting;

    return this;
}
```

### 2. Validate Configuration

```js
function configure( required struct config ) {
    super.configure( arguments.config );

    // Validate required fields
    if ( !arguments.config.keyExists( "apiEndpoint" ) ) {
        throw(
            type: "ConfigurationException",
            message: "CustomVectorMemory requires 'apiEndpoint' in configuration"
        );
    }

    // Validate dimensions match embedding model
    if ( variables.dimensions != getModelDimensions( variables.embeddingModel ) ) {
        throw(
            type: "ConfigurationException",
            message: "Dimensions (#variables.dimensions#) don't match model (#variables.embeddingModel#)"
        );
    }

    return this;
}
```

### 3. Handle Errors Gracefully

```js
function getRelevant( required string query, numeric limit = 5 ) {
    try {
        // Attempt vector search
        var embedding = generateEmbedding( arguments.query );
        return searchVectors( embedding, arguments.limit );

    } catch ( "EmbeddingException" e ) {
        // Log error and return empty results
        writeLog(
            type: "error",
            file: "vector-memory",
            text: "Failed to generate embedding: #e.message#"
        );
        return [];

    } catch ( "VectorSearchException" e ) {
        // Fallback to text search if vector search fails
        writeLog(
            type: "warning",
            file: "vector-memory",
            text: "Vector search failed, using fallback: #e.message#"
        );
        return fallbackTextSearch( arguments.query, arguments.limit );
    }
}
```

### 4. Optimize Embedding Generation

```js
/**
 * Generate embeddings with batching and caching
 */
private function generateEmbeddings( required array texts ) {
    // Batch embed for efficiency
    if ( arguments.texts.len() > 1 ) {
        var result = aiEmbed(
            input: arguments.texts,  // Batch input
            provider: variables.embeddingProvider,
            model: variables.embeddingModel
        );
        return result.embeddings;
    } else {
        var result = aiEmbed(
            input: arguments.texts[ 1 ],
            provider: variables.embeddingProvider,
            model: variables.embeddingModel
        );
        return [ result.embeddings[ 1 ] ];
    }
}
```

### 5. Implement Proper Export/Import

```js
function export() {
    return {
        "type": "custom",
        "class": getMetadata( this ).name,
        "key": variables.key,
        "config": {
            "collection": variables.collection,
            "embeddingProvider": variables.embeddingProvider,
            "embeddingModel": variables.embeddingModel,
            "dimensions": variables.dimensions,
            "metric": variables.metric
        },
        "messages": getAll(),
        "metadata": {
            "count": count(),
            "exportedAt": now()
        }
    };
}

function import( required struct data ) {
    // Validate import data
    if ( !arguments.data.keyExists( "messages" ) ) {
        throw( "Import data must contain 'messages' array" );
    }

    // Clear existing data
    clear();

    // Import messages
    arguments.data.messages.each( msg => {
        add( msg );
    } );

    return this;
}
```

### 6. Monitor Performance

```js
function getRelevant( required string query, numeric limit = 5 ) {
    var startTime = getTickCount();

    try {
        var results = performVectorSearch( arguments.query, arguments.limit );

        var duration = getTickCount() - startTime;

        // Log slow queries
        if ( duration > 1000 ) {  // > 1 second
            writeLog(
                type: "warning",
                file: "vector-memory",
                text: "Slow vector search (#duration#ms) for query: #left(arguments.query, 50)#"
            );
        }

        return results;

    } catch ( any e ) {
        writeLog(
            type: "error",
            file: "vector-memory",
            text: "Vector search failed after #getTickCount() - startTime#ms: #e.message#"
        );
        rethrow;
    }
}
```

### 7. Support Metadata Filtering

```js
function getRelevant(
    required string query,
    numeric limit = 5,
    struct filters = {}
) {
    var embedding = generateEmbedding( arguments.query );

    // Build query with metadata filters
    var searchQuery = {
        "vector": embedding,
        "limit": arguments.limit
    };

    // Add metadata filters
    if ( !arguments.filters.isEmpty() ) {
        searchQuery.filters = arguments.filters;
    }

    return executeSearch( searchQuery );
}
```

