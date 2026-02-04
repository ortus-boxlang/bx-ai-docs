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
  "apiKey": "xai-..."
}
```

**🤗 HuggingFace**:

* **Ollama Server** on `http://localhost:11434`
* **Web UI** on `http://localhost:3000`

#### 🎯 What's Included

The Docker setup provides:

* ✅ **Ollama LLM Server** - Fully configured and ready to use
* ✅ **Web UI** - Browser-based interface for testing and management
* ✅ **Pre-loaded Model** - Automatically downloads `qwen2.5:0.5b-instruct`
* ✅ **Health Checks** - Automatic monitoring and restart capabilities
* ✅ **Persistent Storage** - Data stored locally in `./.ollama` directory
* ✅ **Production Ready** - Configured with proper restart policies
* `mistralai/Mistral-7B-Instruct-v0.3` - Fast and efficient for quick responses

**⚡ Groq**:deploying to production, update these settings in `docker-compose-ollama.yml`:\*\*

1. **Change Default Credentials**

```yaml
   environment:
  "apiKey": "gsk_..."
}
```

**🔷 DeepSeek**:e Models\*\* - Update the preloaded model:

```yaml
command: |
  "ollama serve &
  sleep 10
"apiKey": "sk-..."
}
```

**🟠 Mistral**:

1.  **Add Resource Limits** (recommended):

    ```yaml
    deploy:
      resources:
        limits:
          memory: 8g
          cpus: '4.0'
    ```
2. **SSL/TLS** - Use a reverse proxy (nginx/traefik) for HTTPS

See the comments in `docker-compose-ollama.yml` for complete production setup notes.

#### 📊 Managing the Service

* `mistral-large-latest` - Most capable

**🌐 OpenRouter** (Multi-model gateway): docker compose -f docker-compose-ollama.yml up -d

## View logs

docker compose -f docker-compose-ollama.yml logs -f "apiKey": "sk-or-..." }

```

**🔎 Perplexity**:ces
docker compose -f docker-compose-ollama.yml restart

# Check status
docker compose -f docker-compose-ollama.yml ps
```

}

```

## ⚙️ Configuration Options

### 📊 Settings Reference
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "ollama",
        "apiKey": "",
        "chatURL": "http://localhost:11434",
        "defaultParams": {
          "model": "qwen2.5:0.5b-instruct"
        }
      }
| `returnFormat` | string | `"single"` | Default return format: `single`, `all`, `json`, `xml` or `raw` |

### 🎛️ Default Parameters
```

#### 🌐 Accessing the Web UI

Open your browser to `http://localhost:3000` and login with:

* **Username**: `boxlang` (default)
* **Password**: `rocks` (default)

**⚠️ Change these credentials before production use!**

\| `frequency_penalty` | number | Encourage diversity | `0.1` |

#### 🔐 Environment Variablesconfigurations) is stored in `./.ollama` directory:

```bash
.ollama/
├── server/     # Ollama model data
└── webui/      # Web UI data and settings
```

**Important**: Add `.ollama/` to your `.gitignore` to avoid committing large model files.

#### 🔧 Other Providers

**🔸 Grok (xAI)**:

```json
{
  "provider": "grok",
  "apiKey": "xai-..."
}
```

export OPENAI\_API\_KEY="sk-..."

```

## ✅ Verification
{
  "provider": "huggingface",
  "apiKey": "hf_..."
}
```

**Get your API key**: [https://huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)

**Popular models**:

* `Qwen/Qwen2.5-72B-Instruct` - Default, powerful general-purpose model for complex reasoning
* `meta-llama/Llama-3.1-8B-Instruct` - Meta's Llama model, balanced performance and speed
* `mistralai/Mistral-7B-Instruct-v0.3` - Fast and efficient for quick responses

**Groq**: If configured correctly, you should see a response from your AI provider.

### 🔧 Troubleshooting

#### ❌ "No API key provided"

}

```

**DeepSeek**:
answer = aiChat( "Hello", {}, { provider: "openai", apiKey: "sk-..." } )
```

#### ⏱️ "Connection timeout",

"apiKey": "sk-..." }

````

**Mistral**:

```json
{
  "provider": "mistral",
  "apiKey": "..."
}
````

}

````

### 🦙 Ollama not responding

- `mistral-small-latest` - Cost-effective, fast (default)
- `mistral-medium-latest` - Balanced performance
- `mistral-large-latest` - Most capable

**OpenRouter** (Multi-model gateway):

```json
{
  "provider": "openrouter",
  "apiKey": "sk-or-..."
}
````

### ✅ Verification

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

***

### 🚀 Next Steps

Now that you're installed and configured:

1. [**Provider Setup Guide**](provider-setup.md) - Detailed configuration for all 12+ providers
2. [**Quick Start Guide**](../quickstart.md) - Your first AI conversation in 5 minutes
3. [**Basic Chatting**](../../main-components/chatting/basic-chatting.md) - Learn the fundamentals

#### 💡 Quick Tips

* **Use environment variables** for API keys (never commit to git)
* **Start with Ollama** for free development/testing
* **Try multiple providers** to find what works best for your use case
* **Read the provider guide** for cost comparisons and model recommendations
