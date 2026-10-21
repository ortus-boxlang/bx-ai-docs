---
description: Name a provider, model, params and key once, then select it by name.
icon: layer-group
---

# Profiles

A profile is a named combination of provider, model, params, options and API key. Define `fast` and `planner` once, then pick one per call, per agent, or per run. Without profiles, nothing changes.

## 🛠️ Define

In `boxlang.json`:

```json
{
  "modules": { "bxai": { "settings": {
    "provider": "openai",
    "profiles": {
      "fast": {
        "description": "Cheap and quick, for small tasks",
        "provider": "openai",
        "model": "gpt-4.1-mini",
        "params": { "temperature": 0 },
        "apiKeyEnv": "OPENAI_API_KEY"
      },
      "planner": {
        "description": "Slower and smarter, for planning",
        "provider": "claude",
        "model": "claude-opus-5-5",
        "params": { "temperature": 0.7, "max_tokens": 8192 },
        "apiKeyEnv": "CLAUDE_API_KEY"
      }
    },
    "defaultProfile": ""
  } } }
}
```

| Key | Meaning |
|---|---|
| `description` | Free text for people and UIs. Returned by `aiProfile()` and `aiProfiles()` |
| `provider` | Provider key. Empty uses the top level `provider` |
| `model` | Shorthand for `params.model` |
| `params` | Request params for this profile |
| `options` | Request options for this profile, such as `timeout` |
| `apiKeyEnv` | The **name** of the environment variable that holds the key. Preferred |
| `apiKey` | A literal key. Avoid it |

The profile `default` always exists. It is your top level `provider`, `apiKey` and `defaultParams`. Set `defaultProfile` to make another profile the one used when a call names no provider.

**Rules**, checked when the module starts: a profile cannot be named like a provider (`openai`, `claude`, ...) or `default`, names use letters, numbers, `_` and `-`, and `defaultProfile` must exist. A broken setting throws `InvalidConfiguration`.

## 🎯 Use

```javascript
// Chat and embeddings: options.profile
aiChat( "Summarize this", {}, { profile: "fast" } )
aiChatStream( "Tell a story", chunk => print( chunk ), {}, { profile: "fast" } )
aiEmbed( "Hello", {}, { profile: "embed" } )

// Services, models and agents: the profile argument
service = aiService( profile: "planner" )
model   = aiModel( profile: "fast" )
agent   = aiAgent( name: "support", profile: "fast" )
```

An explicit provider, key, param or option in the call always wins. Params layer from lowest to highest: `defaultParams`, `providers.<name>.params`, the profile, the call. An unknown profile throws `ProfileNotFound`.

{% hint style="info" %}
Pass either `model` or `profile` to `aiAgent()`, not both.
{% endhint %}

## 🔀 Switch an agent

```javascript
// One run only: the agent keeps its own profile, memory is shared
agent.run( "Plan the migration", {}, { profile: "planner" } )

// For good: replaces the model, keeps memory, tools, sub-agents, params and options
agent.withProfile( "planner" )
agent.getProfile() // "planner"
```

`stream()` takes the same `profile` option. To give a sub-agent its own profile, create it with one: `aiAgent( name: "researcher", profile: "planner" )`, then pass it in `subAgents`.

## 📋 List

Built for chat UIs and remote control, such as `/profile` and `/provider` commands:

```javascript
aiProfiles()        // all profiles, the default first
aiProfile( "fast" ) // one profile
aiProviders()       // the providers you can pick
```

See [aiProfiles](../advanced/reference/built-in-functions/aiprofiles.md), [aiProfile](../advanced/reference/built-in-functions/aiprofile.md) and [aiProviders](../advanced/reference/built-in-functions/aiproviders.md).

## 🔐 Keys stay secret

* Use `apiKeyEnv`. The key is read from the environment when the profile is used.
* Listings return `hasApiKey` and the variable name, never the key. Params and options that look like secrets (`key`, `token`, `secret`, `password`) show as `****`.
* A profile key is only sent to the profile's own provider.
* Profiles are never copied into request options, so they do not reach logs or events.

## 📡 Event

`onAIProfileSwitch` fires when an agent changes profile. See [events](../advanced/events.md#70-onaiprofileswitch).
