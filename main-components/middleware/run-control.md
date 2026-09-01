---
description: Internal middleware used by AiAgent cancel and steer operations per thread
icon: sliders
---

# RunControlMiddleware (Internal)

Class: `bxModules.bxai.models.middleware.core.RunControlMiddleware`

This middleware is internal infrastructure for `AiAgent.cancelRun()` and `AiAgent.steerRun()`. It is auto-attached by `AiAgent` and is not intended for direct user construction.

## Purpose

- Tracks a `CancellationToken` per in-flight thread ID
- Applies cancellation and steer input at LLM/tool checkpoints
- Cleans token registry safely when runs complete

## Public Behavior Surface

You normally use the agent API, not this middleware directly:

```javascript
agent.cancelRun( threadId: "thread-123", reason: "Cancelled by operator" )
agent.steerRun( threadId: "thread-123", message: "Use concise answers and no tools" )
```

## Hooks Used

- `beforeAgentRun` (creates token and injects into run options)
- `beforeLLMCall` (checks cancel/steer)
- `beforeToolCall` (checks cancel/steer)
- `afterAgentRun` (compare-and-swap style token cleanup)

## Internal Methods

- `cancel(threadId, reason)`
- `steer(threadId, input)`

Both return boolean indicating whether a currently active run token existed for that thread.

## Notes

- Cancellation and steering are checkpoint-based, not mid-token interruption.
- If there is no active run for a thread ID, cancel/steer are safe no-ops.
- Token cleanup uses guarded remove semantics so concurrent runs sharing a thread ID do not evict each other incorrectly.
