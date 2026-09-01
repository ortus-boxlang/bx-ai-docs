---
description: Use a second model as a security classifier for injection and harmful content
icon: robot
---

# LLMGuardMiddleware

Class: `bxModules.bxai.models.middleware.security.LLMGuardMiddleware`

Uses a second model (judge) to classify inbound and/or outbound content for security risk. Complements heuristic sanitization with semantic classification.

## Features

- Input judging in `beforeLLMCall`
- Optional output judging in `afterLLMCall`
- Configurable fail-open or fail-closed behavior
- Confidence threshold and category gating
- Prompt fencing to isolate untrusted content
- Verdict caching (in-memory static or named BoxLang cache)
- Re-entrancy guard for nested judge calls

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.security.LLMGuardMiddleware(
    judge         : {},
    checkInput    : true,
    checkOutput   : false,
    failMode      : "open",
    threshold     : 0.7,
    categories    : [ "INJECTION", "HARMFUL" ],
    timeout       : 10,
    cacheEnabled  : true,
    cacheName     : "",
    maxCacheSize  : 1000,
    promptTemplate: ""
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `judge` | struct | `{}` | Judge settings: provider/model/apiKey/params/options |
| `checkInput` | boolean | `true` | Judge user content before main model call |
| `checkOutput` | boolean | `false` | Judge model output after call |
| `failMode` | string | `"open"` | On judge failure: `open` (allow) or `closed` (block) |
| `threshold` | numeric | `0.7` | Minimum confidence to act on non-safe verdict |
| `categories` | array | `["INJECTION","HARMFUL"]` | Verdict categories treated as blockable |
| `timeout` | numeric | `10` | Judge timeout in seconds |
| `cacheEnabled` | boolean | `true` | Enable verdict caching |
| `cacheName` | string | `""` | Named BoxLang cache; empty uses static in-memory cache |
| `maxCacheSize` | numeric | `1000` | In-memory cache size cap |
| `promptTemplate` | string | `""` | Custom classifier prompt; empty uses default |

## Hooks Used

- `beforeLLMCall`
- `afterLLMCall`

## Verdict Contract

The judge is expected to return strict JSON:

```json
{ "verdict": "SAFE|INJECTION|HARMFUL", "confidence": 0.0, "reason": "..." }
```

## Example

```javascript
guard = new bxModules.bxai.models.middleware.security.LLMGuardMiddleware(
    judge: {
        provider: "ollama",
        model   : "llama-guard3",
        params  : { temperature: 0, max_tokens: 256 }
    },
    checkInput : true,
    checkOutput: true,
    failMode   : "closed",
    threshold  : 0.8
)

agent = aiAgent( middleware: [ guard ] )
```

## Notes

- Blocking uses `BXAI.SecurityViolation` throws.
- Judge calls are marked internal to avoid recursive security middleware re-entry.
- Invalid `failMode` throws `InvalidFailMode` during construction.
