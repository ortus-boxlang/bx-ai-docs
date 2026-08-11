---
description: >-
  Events fired by conversation memory — message ingestion, vector search, and
  AI-generated summarization.
icon: memory
---

# Memory Events

### onHybridMemoryAdd

Fired when a message is added to a `HybridMemory` instance. Use this to audit, transform, or track messages as they enter hybrid memory.

**When**: Inside `HybridMemory.add()` after the message has been processed

#### Event Arguments

| Argument  | Type        | Description                              |
| --------- | ----------- | ------------------------------------------ |
| `memory`  | `HybridMemory` | The memory instance receiving the message |
| `message` | `AiMessage` | The message being added                  |

#### Example

```javascript
BoxRegisterInterceptor( "onHybridMemoryAdd", function( event ) {
    println( "Hybrid memory received: role=#event.message.getRole()# chars=#event.message.getContent().len()#" )
})
```

***

### onVectorSearch

Fired whenever a semantic search runs against a vector memory store. Use this for observability, caching query results, or logging retrieval quality.

**When**: After `IVectorMemory.getRelevant()` or `findSimilar()` returns results

#### Event Arguments

| Argument  | Type          | Description                                  |
| --------- | -------------- | ----------------------------------------------- |
| `memory`  | `IVectorMemory` | The vector memory instance that was searched |
| `query`   | `String`      | The search query string                      |
| `limit`   | `Numeric`     | Maximum number of results requested          |
| `results` | `Array`       | Array of retrieved documents/messages        |

#### Example

```javascript
BoxRegisterInterceptor( "onVectorSearch", function( event ) {
    println( "Vector search: query='#event.query#' found=#event.results.len()# limit=#event.limit#" )
})
```

***

### onAIMemorySummarize

Fired after a memory instance successfully compresses its history into an AI-generated summary. Available on **every** conversation memory type, not just `SummaryMemory`.

**When**: Inside `BaseMemory.summarize()`, after the summary is stored

| Argument | Type | Description |
| --- | --- | --- |
| `memory` | `IAiMemory` | The memory instance that was summarized |
| `key` | `String` | The memory key |
| `type` | `String` | The memory type name |
| `userId` | `String` | Tenant user id, if set |
| `conversationId` | `String` | Conversation id, if set |
| `messageCount` | `Numeric` | Messages remaining after compression |
| `summaryLength` | `Numeric` | Character length of the generated summary |
| `keepRecent` | `Numeric` | How many recent messages were kept verbatim |
| `summarizedCount` | `Numeric` | How many messages were folded into the summary |

```javascript
BoxRegisterInterceptor( "onAIMemorySummarize", function( event ) {
    println( "Compressed #event.summarizedCount# messages for #event.userId# into #event.summaryLength# chars" )
})
```
