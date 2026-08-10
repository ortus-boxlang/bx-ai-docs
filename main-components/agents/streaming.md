---
description: >-
  Stream agent responses in real-time using callback-based chunk delivery.
icon: wave-pulse
---

# Agent Streaming

Agents support streaming responses, delivering content in real-time chunks as the AI generates them.

## Basic Streaming

```javascript
agent = aiAgent(
    name       : "StreamAgent",
    description: "Streaming assistant"
)

// Accumulate response as it streams
response = ""

agent.stream(
    onChunk: ( chunk ) => {
        content   = chunk.choices?.first()?.delta?.content ?: ""
        response &= content
        print( content )   // Print each chunk immediately
    },
    input: "Write a short story about BoxLang"
)

// response now contains the full text
println( "" )  // New line after stream
```

## Streaming with Parameters

```javascript
agent = aiAgent(
    name  : "Writer",
    params: { temperature: 0.9, max_tokens: 2000 }
)

agent.stream(
    onChunk: ( chunk ) => {
        content = chunk.choices?.first()?.delta?.content ?: ""
        writeOutput( content )
    },
    input  : "Write a blog post about AI in 2026",
    params : { temperature: 0.8 },    // Override params per call
    options: { returnFormat: "single" }
)
```

## Streaming in a Pipeline

Agents implement `IAiRunnable`, so they work in streaming pipelines:

```javascript
pipeline = aiMessage()
    .user( "Write about: ${topic}" )
    .to( agent )

pipeline.stream(
    onChunk: chunk => print( chunk ),
    input  : { topic: "Quantum Computing" }
)
```

## Streaming with Memory

Streaming calls store messages in memory the same as regular calls:

```javascript
agent = aiAgent(
    name  : "ChatBot",
    memory: aiMemory( "window" )
)

// First exchange
agent.run( "My name is Luis" )

// Stream the next response
agent.stream(
    onChunk: chunk => print( chunk.choices?.first()?.delta?.content ?: "" ),
    input  : "What's my name?"
)
// → Streams "Your name is Luis"
```

## Streaming with Per-Call Identity (v3.0+)

```javascript
agent.stream(
    onChunk: chunk => print( chunk.choices?.first()?.delta?.content ?: "" ),
    input  : "Help me with billing",
    options: {
        userId        : "alice",
        conversationId: "conv-001"
    }
)
```

## Resume Streaming (v3.0+)

After a suspension, resume and stream the continuation:

```javascript
// You supply the threadId on the original run
threadId = "deploy-42"

// Agent was suspended by HumanInTheLoopMiddleware
result = agent.run( "Deploy to production", {}, { threadId: threadId } )

if ( result.isSuspended() ) {
    // Later, resume as a stream:
    agent.resumeStream(
        onChunk : chunk => print( chunk.choices?.first()?.delta?.content ?: "" ),
        decision: "approve",
        threadId: threadId
    )
}
```

{% hint style="info" %}
`resumeStream( onChunk, decision, threadId, editedData, decidedBy, reason )` — `onChunk` comes first. Valid decisions are `approve`, `approve_always`, `approve_session`, `reject`, `edit`, and `cancel`.

When a turn requested several tool calls needing approval, they suspend together as one checkpoint; pass an array of per-call decision structs to resolve them individually. Streaming batch approvals are supported on OpenAI and Claude — the two providers with streaming tool-call support today.
{% endhint %}

## Related Pages

* [Getting Started](getting-started.md) — Agent construction and return formats
* [Memory Management](memory.md) — Per-call identity routing in streaming
* [Middleware](middleware.md) — Intercepting stream lifecycle events
* [Pipeline Streaming](../pipelines/streaming.md) — Streaming in full pipelines
