---
description: Heuristic prompt-injection scanning and unicode hygiene for inbound content and tool results
icon: shield-halved
---

# InputSanitizerMiddleware

Class: `bxModules.bxai.models.middleware.security.InputSanitizerMiddleware`

Scans inbound user content and optional tool results for prompt-injection indicators. Applies unicode normalization and invisible-character stripping before model execution.

## Features

- Scans user-role content before LLM call
- Optional scan of tool and MCP results in `wrapToolCall`
- Unicode hygiene via normalization and zero-width stripping
- Actions: `block`, `strip`, `flag`, `log`
- Findings can be stamped to `chatRequest.providerOptions.securityFindings`

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.security.InputSanitizerMiddleware(
    action          : "flag",
    detectors       : [],
    customPatterns  : [],
    normalizeUnicode: true,
    stripZeroWidth  : true,
    scanToolResults : true
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `action` | string | `"flag"` | Handling mode: `block`, `strip`, `flag`, `log` |
| `detectors` | array | `[]` | Detector names to run; empty means all built-ins |
| `customPatterns` | array | `[]` | Additional patterns: `[ { name, regex } ]` |
| `normalizeUnicode` | boolean | `true` | Apply NFKC normalization |
| `stripZeroWidth` | boolean | `true` | Remove zero-width and invisible characters |
| `scanToolResults` | boolean | `true` | Scan tool result strings for indirect injection |

## Hooks Used

- `beforeLLMCall`
- `wrapToolCall`

## Action Semantics

- `block`: throws `BXAI.SecurityViolation` on findings before LLM call; tool-result findings return blocked marker text
- `strip`: removes matched fragments and continues
- `flag`: keeps content, stamps findings in provider options, logs warnings
- `log`: logs warnings only

## Example

```javascript
agent = aiAgent(
    middleware: [
        new bxModules.bxai.models.middleware.security.InputSanitizerMiddleware(
            action        : "block",
            detectors     : [ "instructionOverride", "jailbreak" ],
            customPatterns: [ { name: "internalCodes", regex: "(?i)PROJ-\\d{4}-SECRET" } ]
        )
    ]
)
```

## Notes

- Invalid actions throw `InvalidAction` during construction.
- Message content is mutated in place, including multipart text content sections.
