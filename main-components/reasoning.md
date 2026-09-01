---
description: Enable and read model reasoning (extended thinking) with a single normalized envelope — message.reasoning / delta.reasoning — across every provider that supports it.
icon: brain-circuit
---

# Reasoning

Reasoning-capable models — Claude's extended thinking, OpenAI's `reasoning_effort`, DeepSeek's `reasoning_content`, Ollama's `message.thinking` — could always be **enabled**: `params` passes straight through to the provider body, so provider-specific reasoning params already reached the API. What was missing was reading the reasoning back: every provider spells it differently on the wire, and each one had to be handled by hand.

BoxLang AI normalizes reasoning onto **one envelope, the same shape every provider already normalizes chat responses onto**:

* Synchronous: `choices[].message.reasoning`
* Streaming: `choices[].delta.reasoning`

You read it identically no matter which model is behind the call.

## 🧠 Enabling Reasoning

```javascript
// Claude extended thinking
result = aiChat( "Solve this step by step: a train leaves...", params: {
    thinking: { type: "enabled", budget_tokens: 10000 }
}, options: { returnFormat: "raw" } )

// OpenAI reasoning effort
result = aiChat( "...", params: { reasoning_effort: "high" }, provider: "openai" )
```

`params` is passed straight through to the provider's request body — BoxLang AI does not model reasoning as a capability interface, since it varies per **model** (Sonnet vs. Haiku, `gpt-5` vs. `gpt-4o`), not per provider, which a per-provider interface can't express.

## 📖 Reading Reasoning

```javascript
result    = aiChat( "...", params: { thinking: { type: "enabled", budget_tokens: 5000 } }, options: { returnFormat: "raw" } )
reasoning = result.choices[1].message.reasoning ?: ""   // "" when the model didn't think — absence is normal
answer    = result.choices[1].message.content
```

**Absence is normal, never an error.** A provider or model that didn't think simply omits the key; always guard with `?: ""` rather than assuming it's present.

### Streaming

Reasoning arrives on `delta.reasoning`, ahead of `delta.content` — the model "thinks" before it "speaks":

```javascript
aiChatStream( "Plan a three-step migration...", ( chunk ) => {
    var delta = chunk.choices?.first()?.delta ?: {}
    if ( !isNull( delta.reasoning ) && len( delta.reasoning ) ) {
        print( "🤔 " & delta.reasoning )
    }
    if ( !isNull( delta.content ) && len( delta.content ) ) {
        print( delta.content )
    }
} )
```

## ✅ Provider Coverage

Every chat provider is covered, by one of three routes:

| Route | Providers |
|---|---|
| Inherited from `BaseService` via `super.chat()`/`super.chatStream()` | Grok, Groq, Mistral, DeepSeek, OpenRouter, Perplexity, MiniMax, HuggingFace, Docker Model Runner, OpenAI-Compatible |
| Inherited from `BaseService` directly | Cohere, Gemini |
| Explicit per-wire mapping | Claude (`thinking_delta`), Ollama (`message.thinking`), Claude-on-Bedrock (`delta.thinking`), OpenAI-shaped Bedrock models (`reasoning`/`reasoning_content`) |

`MockService` can script reasoning for offline tests — it emits `reasoning` before `content`, matching real-provider ordering:

```javascript
mock = aiService( "mock" )
mock.script( [ { content: "The answer is 42.", reasoning: "Let me work through this carefully..." } ] )
```

## ⚠️ Important Behaviors

* **Reasoning is never folded into `content`.** Content and reasoning are always kept as separate keys.
* **Reasoning is never persisted to conversation memory.** Only the final answer is added to the assistant turn — otherwise the model's private thinking would be replayed back to it on the next turn as if it had said it aloud.
* **`OutputGuardMiddleware` scans reasoning too**, not just the final answer — a secret named while thinking and never repeated in the answer is still redacted. See [Middleware — Security](middleware.md#security-middleware).
* **Bedrock's native `thinking` blocks are never modified.** AWS requires Claude-on-Bedrock's thinking blocks to be passed back byte-identical within a tool-use turn; any scrubbing happens on a derived copy that never reaches the wire.

## Related Pages

* [Basic Chatting](chatting/basic-chatting.md)
* [Advanced Chatting](chatting/advanced-chatting.md)
* [Streaming](chatting/basic-chatting.md#streaming) · [Agents — Streaming](agents/streaming.md)
* [Middleware — Security](middleware.md#security-middleware)
