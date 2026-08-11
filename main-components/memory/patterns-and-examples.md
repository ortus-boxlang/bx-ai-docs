---
description: >-
  Reusable memory patterns and advanced examples for building conversational
  AI applications with BoxLang AI.
icon: diagram-project
---

# Memory Patterns & Examples

## Memory Patterns

### Pattern 1: Conversation Manager

Encapsulate memory logic in a reusable component:

```java
class {
    property name="memory";
    property name="systemPrompt";

    function init( type = "window", config = {} ) {
        variables.memory = aiMemory( arguments.type, config: arguments.config )
        return this
    }

    function setSystemPrompt( prompt ) {
        variables.systemPrompt = arguments.prompt
        variables.memory.add( aiMessage().system( arguments.prompt ) )
        return this
    }

    function chat( userInput ) {
        variables.memory.add( aiMessage().user( arguments.userInput ) )

        response = aiChat( variables.memory.getAll() )

        variables.memory.add( aiMessage().assistant( response ) )

        return response
    }

    function reset() {
        variables.memory.clear()
        if ( !isNull( variables.systemPrompt ) ) {
            variables.memory.add( aiMessage().system( variables.systemPrompt ) )
        }
        return this
    }

    function export() {
        return variables.memory.export()
    }

    function getHistory() {
        return variables.memory.getAll()
    }
}

// Usage
chatManager = new ConversationManager( "window", { maxMessages: 10 } )
    .setSystemPrompt( "You are a helpful coding tutor" )

println( chatManager.chat( "What is a loop?" ) )
println( chatManager.chat( "Show me an example" ) )
```

### Pattern 2: Multi-User Memory

Modern approach using built-in `userId` and `conversationId` parameters:

```java
// Modern multi-tenant approach
function getUserMemory( userId, conversationId = "" ) {
    return aiMemory( memory: "session",
        key: "chat",
        userId: arguments.userId,
        conversationId: arguments.conversationId,
        config: { maxMessages: 20 }
    )
}

function chat( userId, message, conversationId = "" ) {
    memory = getUserMemory( arguments.userId, arguments.conversationId )
    memory.add( aiMessage().user( arguments.message ) )

    response = aiChat( memory.getAll() )

    memory.add( aiMessage().assistant( response ) )

    return response
}

// Usage - each call is automatically isolated by userId/conversationId
println( chat( "alice", "My name is Alice", "support" ) )
println( chat( "alice", "I need help", "sales" ) )
println( chat( "bob", "My name is Bob", "support" ) )
println( chat( "alice", "What's my name?", "support" ) )  // "Alice" - correct context
println( chat( "bob", "What's my name?", "support" ) )    // "Bob" - isolated

// Alternative: Legacy pattern with manual dictionary (not recommended)
class {
    property name="userMemories" default="{}";

    function getUserMemory( userId ) {
        if ( !variables.userMemories.keyExists( arguments.userId ) ) {
            variables.userMemories[ arguments.userId ] = aiMemory( memory: "window", config: {
                maxMessages: 20
            } )
        }
        return variables.userMemories[ arguments.userId ]
    }
}
```

### Pattern 3: Contextual Memory Switching

Switch memory contexts based on conversation topics:

```java
class {
    property name="memories" default="{}";
    property name="currentContext" default="general";

    function switchContext( context ) {
        variables.currentContext = arguments.context

        if ( !variables.memories.keyExists( arguments.context ) ) {
            variables.memories[ arguments.context ] = aiMemory( memory: "window", config: {
                maxMessages: 10
            } )
        }

        return this
    }

    function chat( message ) {
        memory = variables.memories[ variables.currentContext ]
        memory.add( aiMessage().user( arguments.message ) )

        response = aiChat( memory.getAll() )

        memory.add( aiMessage().assistant( response ) )

        return response
    }
}

// Usage
bot = new ContextualBot()

bot.switchContext( "coding" ).chat( "Explain variables" )
bot.switchContext( "cooking" ).chat( "How do I make pasta?" )
bot.switchContext( "coding" ).chat( "What did we discuss?" )  // Remembers coding context
```

### Pattern 4: Memory with Metadata

Track additional context with metadata:

```java
// Create multi-tenant memory
memory = aiMemory( memory: "window",
    userId: "user123",
    conversationId: "support-456",
    config: { maxMessages: 10 }
)

// Store additional metadata
memory.metadata( {
    sessionId: "session789",
    startTime: now(),
    topic: "customer_support",
    priority: "high"
} )

// Add messages
memory.add( aiMessage().user( "I need help" ) )

// Get metadata
info = memory.metadata()
println( "User: #memory.getUserId()#, Topic: #info.topic#" )

// Export with metadata AND tenant identifiers
export = memory.export()
// Contains: { userId: "user123", conversationId: "support-456", messages: [...], metadata: {...} }

// Import preserves tenant identifiers
newMemory = aiMemory( memory: "window", config: { maxMessages: 10 } )
newMemory.import( export )
println( newMemory.getUserId() )  // "user123"
println( newMemory.getConversationId() )  // "support-456"
```

### Pattern 5: Memory Summarization

`summarize( config )` is a method on **every** conversation memory type — `WindowMemory`, `SummaryMemory`, `CacheMemory`, `FileMemory`, `JdbcMemory`, `SessionMemory`, `HybridMemory` — not just `SummaryMemory`. Call it any time to explicitly condense old messages, regardless of whether the memory auto-triggers compression.

```java
memory = aiMemory( memory: "window", config: { maxMessages: 50 } )

// ... have a long conversation ...

// Explicitly compress now, with per-call overrides
memory.summarize( {
    keepRecent: 5,               // messages to keep verbatim (defaults to summaryThreshold)
    model     : "gpt-4o-mini",   // overrides the instance's summaryModel for this call
    provider  : "openai"
} )
```

Persistent stores (`JdbcMemory`, `FileMemory`, `CacheMemory`) automatically persist the compressed result. Vector memories are semantic indexes, not conversation buffers, so `summarize()` is a no-op there.

`SummaryMemory` still auto-triggers this on its own `maxMessages`/`maxTokens` threshold — see [above](README.md#summary-memory) — but you're no longer limited to that one memory type for on-demand compression.

{% hint style="info" %}
`onAIMemorySummarize` fires after every successful summarization, on any memory type — see [Event System](../../advanced/events/memory-events.md#onaimemorysummarize).
{% endhint %}

***


## Advanced Examples

### Example 1: RAG with Memory

Combine retrieval-augmented generation with conversation memory:

```java
// Multi-tenant RAG system
function chatWithKnowledge( userId, conversationId, userQuery ) {
    // Create user-specific memory
    memory = aiMemory( memory: "session",
        userId: arguments.userId,
        conversationId: arguments.conversationId,
        config: { key: "rag", maxMessages: 10 }
    )

    // Retrieve relevant documents
    relevantDocs = searchDocuments( userQuery )

    // Build context
    context = "Relevant information:\n" & relevantDocs.toList()

    // Add to memory
    memory.add( aiMessage().system( context ) )
    memory.add( aiMessage().user( userQuery ) )

    // Generate response
    response = aiChat( memory.getAll() )

    memory.add( aiMessage().assistant( response ) )

    return response
}

// Usage with automatic isolation
response1 = chatWithKnowledge( "alice", "research-1", "What is quantum computing?" )
response2 = chatWithKnowledge( "alice", "research-2", "What is machine learning?" )
response3 = chatWithKnowledge( "bob", "research-1", "What is quantum computing?" )
// All three conversations are isolated
```

### Example 2: Multi-Stage Memory Pipeline

```java
// Stage 1: Collect information
infoMemory = aiMemory( memory: "window", config: { maxMessages: 5 } )
infoMemory.add( aiMessage().system( "Collect user requirements" ) )

// ... gather requirements ...

// Stage 2: Generate solution using collected info
solutionMemory = aiMemory( memory: "window", config: { maxMessages: 10 } )
solutionMemory.add( aiMessage().system( "Generate solution based on requirements" ) )

// Transfer relevant context
summary = summarizeConversation( infoMemory )
solutionMemory.add( aiMessage().user( "Requirements: #summary#" ) )

// Generate solution
solution = aiChat( solutionMemory.getAll() )
```

### Example 3: Adaptive Memory

Adjust memory size based on conversation complexity:

```java
class {
    property name="memory";
    property name="baseLimit" default="10";

    function init() {
        variables.memory = aiMemory( memory: "window", config: { maxMessages: variables.baseLimit } )
        return this
    }

    function chat( message ) {
        variables.memory.add( aiMessage().user( message ) )

        // Detect if conversation is getting complex
        if ( isComplexQuery( message ) ) {
            // Increase memory limit temporarily
            expandMemory( variables.baseLimit * 2 )
        }

        response = aiChat( variables.memory.getAll() )

        variables.memory.add( aiMessage().assistant( response ) )

        return response
    }

    function isComplexQuery( message ) {
        keywords = [ "explain", "detailed", "comprehensive", "analyze" ]
        return keywords.some( kw => message.findNoCase( kw ) > 0 )
    }

    function expandMemory( newLimit ) {
        // Create new memory with larger limit
        oldMessages = variables.memory.getAll()
        variables.memory = aiMemory( memory: "window", config: { maxMessages: newLimit } )
        oldMessages.each( msg => variables.memory.add( msg ) )
    }
}
```

***

