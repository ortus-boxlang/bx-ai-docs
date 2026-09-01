---
description: Redact secrets and strip exfiltration channels from model outputs and reasoning
icon: shield
---

# OutputGuardMiddleware

Class: `bxModules.bxai.models.middleware.security.OutputGuardMiddleware`

Guards outbound model content by redacting sensitive values and removing exfiltration markdown channels.

## Features

- Redacts secrets and PII patterns with configurable mask
- Strips markdown image/link exfiltration patterns with host allowlists
- Scans reasoning content as well as final answer content
- Actions: `redact`, `flag`, `block`
- Optional scan of final agent output in `afterAgentRun`

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.security.OutputGuardMiddleware(
    action             : "redact",
    redactors          : [],
    customRedactors    : {},
    mask               : "[REDACTED]",
    stripMarkdownImages: true,
    stripExternalLinks : false,
    allowedImageHosts  : [],
    allowedLinkHosts   : [],
    scanAgentOutput    : true
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `action` | string | `"redact"` | Behavior on findings: `redact`, `flag`, `block` |
| `redactors` | array | `[]` | Named redactor set; empty uses default redactors |
| `customRedactors` | struct | `{}` | Extra redactors as regex or closure mappers |
| `mask` | string | `"[REDACTED]"` | Replacement text for redacted values |
| `stripMarkdownImages` | boolean | `true` | Strip non-allowlisted markdown images |
| `stripExternalLinks` | boolean | `false` | Rewrite external links to visible text |
| `allowedImageHosts` | array | `[]` | Allowlist for image hostnames |
| `allowedLinkHosts` | array | `[]` | Allowlist for link hostnames |
| `scanAgentOutput` | boolean | `true` | Also scan final output in `afterAgentRun` |

## Hooks Used

- `afterLLMCall`
- `afterAgentRun`

## Action Semantics

- `redact`: mutates resolved response fields to scrub detected data
- `flag`: preserves response, logs and records findings
- `block`: throws `BXAI.SecurityViolation`

## Example

```javascript
agent = aiAgent(
    middleware: [
        new bxModules.bxai.models.middleware.security.OutputGuardMiddleware(
            action            : "redact",
            mask              : "[MASKED]",
            stripExternalLinks: true,
            allowedLinkHosts  : [ "docs.ortusbooks.com" ]
        )
    ]
)
```

## Notes

- Invalid actions throw `InvalidAction` at construction.
- Streaming is detection-oriented: `afterLLMCall` runs once the stream has already emitted chunks.
- String-only agent responses cannot always be rewritten in `afterAgentRun`; redaction reliability is strongest at `afterLLMCall`.
