---
description: Block or reject tool invocations by tool name and argument regex policies
icon: ban
---

# GuardrailMiddleware

Class: `bxModules.bxai.models.middleware.core.GuardrailMiddleware`

Guards tool execution only. It can block specific tools or reject calls whose arguments match forbidden regex patterns.

## Features

- Denylist tool names with `blockedTools`
- Validate arguments per tool with `argPatterns`
- Works with both normalized `toolArgs` and provider-native tool call shapes
- Returns `reject()` with reason when a rule matches

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.core.GuardrailMiddleware(
    blockedTools: [ "deleteRecord", "dropTable" ],
    argPatterns : {
        runSQL: [ { query: "^SELECT" } ]
    }
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `blockedTools` | array | `[]` | Tool names that are always rejected |
| `argPatterns` | struct | `{}` | Per-tool regex rules in the form `{ toolName: [ { paramName: "pattern" } ] }` |

## Hooks Used

- `beforeToolCall`

## Example

```javascript
agent = aiAgent(
    tools      : [ runSQLTool, deleteUserTool ],
    middleware : [
        new bxModules.bxai.models.middleware.core.GuardrailMiddleware(
            blockedTools: [ "deleteUser" ],
            argPatterns : {
                runSQL: [
                    { query: "(?i)^\\s*select\\b" },
                    { query: "(?i)drop|truncate|delete" }
                ]
            }
        )
    ]
)
```

## Notes

- Tool-name matching for `blockedTools` is case-insensitive.
- Pattern rules run only for tools present in `argPatterns`.
- This middleware does not sanitize prompt text or model output.
