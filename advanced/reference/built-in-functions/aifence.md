---
description: Wraps untrusted content in tamper-resistant boundary markers so the model treats it as data, never instructions.
icon: shield-halved
---

# aiFence

Wraps untrusted content in tamper-resistant boundary markers so the model treats it as **data**, never instructions — the core defense against indirect prompt injection from RAG documents, tool/MCP output, and web pages.

## Syntax

```javascript
aiFence( content, label, withPreamble )
```

## Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `content` | `any` | ✅ | — | The untrusted content to fence (string or complex value) |
| `label` | `string` | ❌ | `"external"` | A short source label, e.g. `knowledge-base`, `web-page` |
| `withPreamble` | `boolean` | ❌ | `false` | Prepend the security preamble to the fenced block |

## Returns

The fenced string.

## How It Works

The content is wrapped in boundary markers carrying a **random per-call id**, and any marker syntax inside the content is neutralized — so an attacker cannot forge a closing marker to "break out" of the fence and have the rest of their text read as instructions.

```
[UNTRUSTED-DATA id=<random> type=knowledge-base]
...content...
[/UNTRUSTED-DATA id=<random>]
```

## Examples

### Fencing Retrieved Context

```javascript
context = aiFence( retrievedDoc, "knowledge-base" )
answer  = aiChat( "Answer using this context: #context#" )
```

### One-Off Prompts Without a System Message

When there's no system message to carry the security preamble, include it inline:

```javascript
fenced = aiFence( untrustedText, "web-page", withPreamble = true )
answer = aiChat( fenced )
```

### Fencing Tool Output

```javascript
searchTool = aiTool( "search", "Search the web", ( query ) => {
    var results = aiWebSearch( query )
    return aiFence( results, "web-search" )
} )
```

### On a Message

```javascript
message = aiMessage()
    .user( "Summarize the attached policy" )
    .addUntrusted( policyDocument, "policy-doc" )
```

## Automatic Fencing

You often don't need to call this yourself. The `${context}` render path is **auto-fenced by default** for every `aiChat()` / `aiModel()` / `aiAgent()` request that passes context:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "security": {
          "fencing": {
            "enabled"       : true,
            "fenceContext"  : true,
            "escapeBindings": true,
            "preamble"      : ""
          }
        }
      }
    }
  }
}
```

Requests that pass no context are unaffected. Opt out globally with `security.fencing.enabled = false`, per request with `secure: false`, or per message with `aiMessage().setContextTrust( true )`.

{% hint style="info" %}
Fencing is on by default even when `settings.security.enabled` is `false` — it only changes requests that actually pass untrusted context, and the protection is close to free.
{% endhint %}

## Related

* [Security Guide](../../../deployment/security/README.md) — the full guardrail stack
* [Message Context](../../../main-components/messages/message-context.md)
* [RAG](../../../rag/rag.md)
