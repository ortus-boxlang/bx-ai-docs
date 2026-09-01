---
description: Approve, reject, edit, defer, and suspend tool calls with policy- and gateway-driven human approval
icon: user-check
---

# HumanInTheLoopMiddleware

Class: `bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware`

Adds human approval to tool calls using three collaborators:

- `IApprovalPolicy` decides if a tool call needs approval
- `IGateway` presents and resolves the decision
- `HumanInteractionCoordinator` tracks suspensions and decisions

## Features

- Tool-name approval policy by default
- Optional callback-based or custom policy-based approval
- CLI mode with default `CliGateway`
- Web mode with deferred suspension
- Gateway mode for platform-backed approvals
- Batch-aware defer/suspend via `afterToolBatch`
- Resume-aware behavior (`agent.resume`) with approve/reject/edit mapping
- Durable grant support via `IDecisionStore`

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware(
    toolsRequiringApproval: [ "sendEmail" ],
    mode                  : "cli",
    showArguments         : true,
    approvalCallback      : function( context ){ return true; },
    policy                : customPolicy,
    gateway               : aiGateway( "http" ),
    decisionStore         : aiDecisionStore( "jdbc", { datasource: "myDSN" } )
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `toolsRequiringApproval` | array | `[]` | Tool names requiring approval when no custom policy is provided |
| `mode` | string | `"cli"` | Approval mode without explicit gateway: `"cli"` or `"web"` |
| `showArguments` | boolean | `true` | Include tool arguments in approval prompt/message |
| `approvalCallback` | function | none | Function policy fallback: returns true when approval is required |
| `policy` | `IApprovalPolicy` | `ToolNameApprovalPolicy` | Explicit policy override |
| `gateway` | `IGateway` | `CliGateway` unless `mode="web"` | Gateway used for human interaction |
| `decisionStore` | `IDecisionStore` | `aiDecisionStore()` | Durable decision/grant store |

## Hooks Used

- `onAttach`
- `beforeToolCall`
- `afterToolBatch`

## Mode Behavior

### CLI mode

- Auto-attaches `CliGateway` when no explicit gateway is provided
- Blocks for immediate terminal decision

### Web mode

- No default gateway
- Uses `defer()` on individual tool calls and `suspend()` once per turn in `afterToolBatch`
- Requires an agent checkpointer to resume safely

### Explicit gateway

- Uses provided gateway regardless of mode
- Asynchronous gateways return suspensions with suspension IDs

## Resume Semantics

When resuming, middleware maps human decisions to middleware outcomes:

- approve -> `AiMiddlewareResult.approve()`
- reject -> `AiMiddlewareResult.reject(reason)`
- edit -> mutates provider-specific tool-call arguments, then `continue()`
- cancel/other -> `AiMiddlewareResult.cancel(reason)`

## Example

```javascript
import bxModules.bxai.models.middleware.core.HumanInTheLoopMiddleware;

agent = aiAgent(
    tools       : [ deployTool ],
    checkpointer: aiMemory( "cache" ),
    middleware  : [
        new HumanInTheLoopMiddleware(
            mode                  : "web",
            toolsRequiringApproval: [ "deploy" ],
            gateway               : aiGateway( "http" )
        )
    ]
)
```

## Notes

- If suspension is possible and the agent has no checkpointer, middleware throws on attach.
- Unknown `mode` values fall back to CLI gateway with a deprecation warning.
- `afterToolBatch` enables batched suspension so multiple pending decisions can pause as one checkpoint.
