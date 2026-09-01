---
description: Quick installation guide for BoxLang AI module.
icon: download
---

# installation

## 📦 Installation

Get the BoxLang AI module installed and ready to use in minutes.

### 📑 Table of Contents

* [System Requirements](./#-system-requirements)
* [Installation Methods](./#-installation-methods)
* [Module Configuration](./#-module-configuration)
* [Running Ollama with Docker](./#-running-ollama-with-docker)
* [Verification](./#-verification)
* [Next Steps](./#-next-steps)

### ⚙️ System Requirements

* **BoxLang Runtime**: 1.8+
* **Internet**: Required for cloud providers (OpenAI, Claude, etc.)
* **Optional**: Docker for running Ollama locally

### 🚀 Installation Methods

#### 📥 BoxLang Module Installer

The simplest way to install the module is via the BoxLang Module Installer globally:

```bash
install-bx-module bx-ai
```

This command downloads and installs the module globally, making it available to all BoxLang applications on your system. If you want to install it locally in your cli or other runtimes:

```bash
install-bx-module bx-ai --local
```

#### 📦 CommandBox Package Manager

For CommandBox-based web applications and runtimes

```bash
box install bx-ai
```

This adds the module to your application's dependencies and installs it in the appropriate location.

#### 📋 Application Dependencies

Add to your `box.json` for managed dependencies:

```json
{
  "name": "my-boxlang-app",
  "version": "1.0.0",
  "dependencies": {
    "bx-ai": "^2"
  }
}
```

Then run:

```bash
box install
```

### 🔧 Module Configuration

Set up your first AI provider in `boxlang.json`:

#### Basic Setup (OpenAI)

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openai",
        "apiKey": "sk-your-key-here"
      }
    }
  }
}
```

#### Using Environment Variables (Recommended)

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openai",
        "apiKey": "${OPENAI_API_KEY}"
      }
    }
  }
}
```

Then set the environment variable:

```bash
export OPENAI_API_KEY="sk-..."
```

#### Local AI (Ollama)

For free, local AI with no API costs:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "ollama",
        "chatURL": "http://localhost:11434",
        "defaultParams": {
          "model": "llama3.2"
        }
      }
    }
  }
}
```

#### Predefined Providers (v2.1.0+)

Configure multiple providers with default parameters and service options:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openai",
        "apiKey": "${OPENAI_API_KEY}",
        "providers": {
          "openai": {
            "params": {
              "model": "gpt-4"
            },
            "options": {
              "apiKey": "${OPENAI_API_KEY}"
            }
          },
          "ollama": {
            "params": {
              "model": "qwen2.5:0.5b-instruct"
            },
            "options": {
              "baseURL": "http://my-ollama-server:11434"
            }
          },
          "claude": {
            "params": {
              "model": "claude-3-5-sonnet-20241022"
            },
            "options": {
              "apiKey": "${CLAUDE_API_KEY}",
              "timeout": 120
            }
          }
        }
      }
    }
  }
}
```

**Benefits of predefined providers:**

- ✅ **Centralized configuration** - All provider settings in one place
- ✅ **Default parameters** - Set model, temperature, and other params per provider
- ✅ **Service options** - Configure custom endpoints, timeouts, headers, and API keys
- ✅ **Environment-specific** - Easy to override for dev/staging/production
- ✅ **Simplified code** - Just reference provider name: `aiModel("ollama")`

When you use `aiModel("ollama")`, it automatically applies the predefined `params` and `options` for that provider.

**📖 For detailed provider setup, see** [**Provider Setup Guide**](provider-setup.md)

***

### 🐳 Running Ollama with Docker

For production deployments or easier setup, use the included Docker Compose configuration:

#### 📋 Quick Start

```bash
docker compose -f docker-compose-ollama.yml up -d
```

This starts:

* **Ollama Server** on `http://localhost:11434`
* **Web UI** on `http://localhost:3000`

#### 🎯 What's Included

The Docker setup provides:

* ✅ **Ollama LLM Server** - Fully configured and ready to use
* ✅ **Web UI** - Browser-based interface for testing and management
* ✅ **Pre-loaded Models** - Automatically pulls `qwen3:0.6b`, `gemma3`, and `nomic-embed-text`
* ✅ **Health Checks** - Automatic monitoring and restart capabilities
* ✅ **Persistent Storage** - Data stored locally in `./.ollama` directory
* ✅ **Production Ready** - Configured with proper restart policies

#### 📊 Managing the Service

```bash
# View logs
docker compose -f docker-compose-ollama.yml logs -f

# Restart services
docker compose -f docker-compose-ollama.yml restart

# Check status
docker compose -f docker-compose-ollama.yml ps
```

#### 🌐 Accessing the Web UI

Open your browser to `http://localhost:3000` and login with:

* **Email**: `admin@boxlang.io` (default)
* **Password**: `rocks` (default)

**⚠️ Change these credentials before production use!**

#### 🔐 Persistent Storage

All Ollama and Web UI data (models, chat history, configurations) is stored in `./.ollama`:

```bash
.ollama/
├── server/     # Ollama model data
└── webui/      # Web UI data and settings
```

**Important**: Add `.ollama/` to your `.gitignore` to avoid committing large model files.

#### ⚙️ Before Deploying to Production

Update these settings in `docker-compose-ollama.yml`:

1. **Change Default Credentials** — set strong `WEBUI_ADMIN_USER`/`WEBUI_ADMIN_PASS` (or `WEBUI_EMAIL`/`WEBUI_PASSWORD`) values, ideally via Docker secrets or an env file rather than committed to the compose file.
2. **Choose Production Models** — update the `ollama pull` lines in the `command:` block to the models you actually need.
3. **Add Resource Limits** (recommended):

   ```yaml
   deploy:
     resources:
       limits:
         memory: 8g
         cpus: '4.0'
   ```
4. **SSL/TLS** - Use a reverse proxy (nginx/traefik) for HTTPS.

See the comments in `docker-compose-ollama.yml` for complete production setup notes.

***

## ⚙️ Configuration Options

### 📊 Settings Reference

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "ollama",
        "apiKey": "",
        "chatURL": "http://localhost:11434",
        "defaultParams": {
          "model": "qwen3:0.6b"
        }
      }
    }
  }
}
```

| Setting | Type | Default | Description |
|---|---|---|---|
| `provider` | string | `"openai"` | Default provider used when none is passed per-call |
| `apiKey` | string | `""` | Default API key for `provider` |
| `defaultParams` | struct | `{}` | Default request params merged into every call (e.g. `model`, `temperature`) |
| `providers` | struct | `{}` | Per-provider `params`/`options` overrides — see [Predefined Providers](#predefined-providers-v210) above |
| `timeout` | numeric | `90` | Default AI request timeout, in seconds |
| `returnFormat` | string | `"single"` | Default return format: `single`, `all`, `json`, `xml` or `raw` |
| `logRequest` / `logResponse` | boolean | `false` | Log AI requests/responses to `ai.log` |
| `logRequestToConsole` / `logResponseToConsole` | boolean | `false` | Log AI requests/responses to the console |

This is the quick-reference subset. See the [**Provider Setup Guide**](provider-setup.md#configuration-options-reference) for the full settings reference, including `memory`, `audio`, `image`, `security`, `hitl`, and `gateways`.

### 🎛️ Default Parameters

`defaultParams` accepts any parameter the target provider's chat API accepts — commonly `model`, `temperature`, `max_tokens`, `top_p`, and `frequency_penalty`:

```json
{
  "defaultParams": {
    "model": "gpt-4o",
    "temperature": 0.7,
    "max_tokens": 1000,
    "frequency_penalty": 0.1
  }
}
```

***

## ✅ Verification

Test your installation:

```javascript
// test-ai.bxs
answer = aiChat( "Say hello!" )
println( answer )
```

Run it:

```bash
boxlang test-ai.bxs
```

If configured correctly, you should see a response from your AI provider.

### 🔧 Troubleshooting

#### ❌ "No API key provided"

Set the API key for your configured provider, either in `boxlang.json` or by passing it per-call:

```javascript
answer = aiChat( "Hello", {}, { provider: "openai", apiKey: "sk-..." } )
```

#### ⏱️ "Connection timeout"

Increase `settings.timeout` (default `90` seconds) or the per-request `options.timeout` if you're on a slow network or using a large/local model:

```javascript
answer = aiChat( "Hello", {}, { timeout: 120 } )
```

#### 🦙 Ollama not responding

* Confirm the container is healthy: `docker compose -f docker-compose-ollama.yml ps`
* Confirm the model is pulled: `docker exec ollama ollama list`
* Confirm `chatURL` points at the right host/port (`http://localhost:11434` by default)

***

### 🚀 Next Steps

Now that you're installed and configured:

1. [**Provider Setup Guide**](provider-setup.md) - Detailed configuration for all supported providers
2. [**Quick Start Guide**](../quickstart.md) - Your first AI conversation in 5 minutes
3. [**Basic Chatting**](../../main-components/chatting/basic-chatting.md) - Learn the fundamentals

#### 💡 Quick Tips

* **Use environment variables** for API keys (never commit to git)
* **Start with Ollama** for free development/testing
* **Try multiple providers** to find what works best for your use case
* **Read the provider guide** for cost comparisons and model recommendations
