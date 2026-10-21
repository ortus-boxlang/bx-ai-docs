---
description: List the providers you can select. Never returns an API key.
icon: server
---

# aiProviders

List the providers you can select. Never returns an API key.

## Syntax

```javascript
aiProviders()
```

## Parameters

No parameters.

## Returns

An array of structs: `name`, `isDefault` and `hasApiKey` (true when `<PROVIDER>_API_KEY` is set, or the global `apiKey` is set for the default provider). It lists the built in providers plus any named in the `providers` setting.

## Examples

```javascript
aiProviders().filter( provider => provider.hasApiKey ).each( provider => println( provider.name ) )
```

See [Profiles](../../../main-components/profiles.md).
