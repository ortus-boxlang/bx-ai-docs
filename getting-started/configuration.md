---
description: The bx-ai settings you will actually use, on one page.
icon: gear
---

# Configuration

Settings live in `boxlang.json` under `modules.bxai.settings`. Every setting has a default, so set only what you need. Your values are merged over the defaults.

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openai",
        "defaultParams": { "model": "gpt-4.1", "temperature": 0.7 }
      }
    }
  }
}
```

## The basics

| Setting | Default | What it does |
|---|---|---|
| `provider` | `"openai"` | Provider used when a call does not name one |
| `apiKey` | `""` | Key for `provider`. Prefer the `<PROVIDER>_API_KEY` env var |
| `defaultParams` | `{}` | Request params for every call, such as `model` and `temperature` |
| `timeout` | `90` | Request timeout in seconds |
| `returnFormat` | `"single"` | What `aiChat()` returns: `single`, `all`, `raw`, `json` or `xml` |

## Providers

Give a provider its own `params` and `options`. They apply whenever that provider is used:

```json
"providers": {
  "openai": { "params": { "model": "gpt-4.1" } },
  "ollama": { "params": { "model": "qwen3:0.6b" }, "options": { "baseURL": "http://my-server:11434" } }
}
```

## Profiles

A profile is a named combination of provider, model, params, options and key. Use it when you switch between setups, such as a `fast` model and a `planner` model:

```json
"profiles": {
  "fast":    { "description": "Cheap and quick", "provider": "openai", "model": "gpt-4.1-mini", "params": { "temperature": 0 }, "apiKeyEnv": "OPENAI_API_KEY" },
  "planner": { "provider": "claude", "model": "claude-opus-5-5", "params": { "max_tokens": 8192 }, "apiKeyEnv": "CLAUDE_API_KEY" }
}
```

```javascript
aiChat( "Summarize this", {}, { profile: "fast" } )
```

See [Profiles](../main-components/profiles.md) for agents, switching and listing.

## Embeddings

`aiEmbed()` uses your chat provider unless you say otherwise:

```json
"embeddings": { "defaultProvider": "openai", "defaultModel": "text-embedding-3-small" }
```

## Logging

| Setting | Default | What it does |
|---|---|---|
| `logRequest` / `logResponse` | `false` | Write requests or responses to `ai.log` |
| `logRequestToConsole` / `logResponseToConsole` | `false` | Print them to the console. Best way to debug a provider |

## Everything else

You can leave these alone until you need them.

| Setting | Learn more |
|---|---|
| `memory` | [Memory](../main-components/memory/README.md) |
| `audio` | [Audio](../main-components/audio/README.md) |
| `image` | [Image generation](../main-components/image-generation/README.md) |
| `security` | [Security](../deployment/security.md) |
| `webSearch` | [Web search](../main-components/web-search/README.md) |
| `mcp` | [MCP stdio clients](../mcp/client/stdio.md), [MCP server rate limiting](../mcp/server/rate-limiting.md#module-settings) and trusted proxies |
| `skillsDirectory`, `autoLoadSkills` | [Skills](../main-components/skills.md) |
| `gateways` | [Gateways](../main-components/gateways.md) |
| `hitl` | [Human-in-the-Loop](../main-components/human-in-the-loop.md) |

## API keys

1. Set `<PROVIDER>_API_KEY` in the environment, for example `OPENAI_API_KEY`. This is the default way and needs no setting.
2. For a profile, set `apiKeyEnv` to the **name** of the variable. The profile never stores the key.
3. Pass `apiKey` in the call options to override everything else.

## Override per call

A call always wins over settings:

```javascript
aiChat( "Hello", { temperature: 0 }, { provider: "claude", timeout: 120 } )
```

The order, lowest to highest: `defaultParams`, the provider's `params`, the profile, the params of the call.

Full provider walkthroughs are in [Provider Setup](installation/provider-setup.md).
