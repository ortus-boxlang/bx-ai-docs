---
description: Install the BoxLang AI module and make your first call.
icon: download
---

# Installation

## 📦 Install

**Requirements**: BoxLang 1.8+, and either an API key for a cloud provider or [Ollama](provider-setup.md) for free local AI.

Install the module globally with the BoxLang module installer:

```bash
install-bx-module bx-ai
```

Or locally for one application or runtime:

```bash
install-bx-module bx-ai --local
```

Using CommandBox:

```bash
box install bx-ai
```

## 🔑 Set your first provider

Put your key in an environment variable and name the provider in `boxlang.json`:

```bash
export OPENAI_API_KEY="sk-..."
```

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openai"
      }
    }
  }
}
```

The module reads `<PROVIDER>_API_KEY` for you, so `OPENAI_API_KEY` is picked up without an `apiKey` setting. Never commit a key to git.

## ✅ Verify

```javascript
// test-ai.bxs
println( aiChat( "Say hello!" ) )
```

```bash
boxlang test-ai.bxs
```

You should see a reply from your provider. If you get an error, check that the key is set in the same shell and see the troubleshooting section of [Provider Setup](provider-setup.md).

## 🚀 Next steps

1. [**Configuration**](../configuration.md): the settings you will use, on one page
2. [**Provider Setup**](provider-setup.md): details for every provider
3. [**Quick Start**](../quickstart.md): your first conversation
