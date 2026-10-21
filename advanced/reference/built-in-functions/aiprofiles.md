---
description: List the configured profiles. Never returns an API key.
icon: layer-group
---

# aiProfiles

List the configured profiles. Never returns an API key.

## Syntax

```javascript
aiProfiles()
```

## Parameters

No parameters.

## Returns

An array of structs, the default profile first: `name`, `provider`, `model`, `params`, `options`, `apiKeyEnv`, `hasApiKey`, `isDefault`. Secret looking params and options are masked as `****`.

## Examples

```javascript
profiles = aiProfiles()
profiles.each( profile => println( "#profile.name# -> #profile.provider# #profile.model#" ) )
```

See [Profiles](../../../main-components/profiles.md).
