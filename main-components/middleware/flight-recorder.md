---
description: Record and replay LLM and tool interactions to deterministic JSON fixtures
icon: cassette-tape
---

# FlightRecorderMiddleware

Class: `bxModules.bxai.models.middleware.core.FlightRecorderMiddleware`

Records structured LLM/tool interactions to fixture files and can replay them without live provider or tool execution.

## Features

- Three modes: `passthrough`, `record`, `replay`
- Captures LLM requests/responses and optional tool args/results
- Incremental snapshot writes for crash-safe recordings
- Strict or lenient replay matching
- Public inspection helpers: `getTape()`, `getEffectiveFixturePath()`, `reset()`

## Constructor

```javascript
middleware = new bxModules.bxai.models.middleware.core.FlightRecorderMiddleware(
    mode       : "record",
    fixturePath: "",
    fixtureDir : "/.agents/flight-recorder",
    recordTools: true,
    strict     : true
)
```

## Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | string | `"passthrough"` | Operating mode: `passthrough`, `record`, `replay` |
| `fixturePath` | string | `""` | Explicit fixture path (required in replay mode) |
| `fixtureDir` | string | `"/.agents/flight-recorder"` | Directory used when auto-generating fixture files |
| `recordTools` | boolean | `true` | Record/replay tool interactions |
| `strict` | boolean | `true` | Enforce exact interaction type sequence during replay |

## Hooks Used

- `beforeAgentRun`
- `afterAgentRun`
- `wrapLLMCall`
- `wrapToolCall`

## Example

```javascript
agent = aiAgent(
    name      : "weather-agent",
    middleware: [
        new bxModules.bxai.models.middleware.core.FlightRecorderMiddleware(
            mode       : "record",
            fixtureDir : "/.agents/flight-recorder",
            recordTools: true
        )
    ]
)
```

```javascript
agent = aiAgent(
    middleware: [
        new bxModules.bxai.models.middleware.core.FlightRecorderMiddleware(
            mode       : "replay",
            fixturePath: "tests/fixtures/weather-agent.json",
            strict     : true
        )
    ]
)
```

## Notes

- In replay mode, missing fixture files throw `FlightRecorder.FixtureNotFound`.
- Strict mode throws on interaction-type mismatch; lenient mode scans forward for next matching type.
- Auto-generated file names are normalized from agent name plus timestamp.
