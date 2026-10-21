---
description: Describe one profile. Never returns an API key.
icon: layer-group
---

# aiProfile

Describe one profile. Never returns an API key.

## Syntax

```javascript
aiProfile( name = "" )
```

## Parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `name` | string | No | `""` | The profile name. Empty is the default profile |

## Returns

A struct with `name`, `provider`, `model`, `params`, `options`, `apiKeyEnv`, `hasApiKey` and `isDefault`. Throws `ProfileNotFound` if the profile does not exist.

## Examples

```javascript
fast = aiProfile( "fast" )
println( fast.provider )
println( fast.hasApiKey )
```

See [Profiles](../../../main-components/profiles.md).
