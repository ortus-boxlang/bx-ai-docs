---
description: >-
  Comprehensive guide to configuring AI providers in BoxLang AI - from API keys
  to local deployment with Ollama.
icon: puzzle-piece
---

# Provider Setup & Configuration

This guide covers detailed setup instructions for all supported AI providers, helping you choose the right provider and configure it properly for your use case.

## 📋 Table of Contents

* [📋 Table of Contents](provider-setup.md#-table-of-contents)
* [🎯 Quick Provider Comparison](provider-setup.md#-quick-provider-comparison)
  * [💡 Recommendations by Use Case](provider-setup.md#-recommendations-by-use-case)
* [🔧 Configuration Basics](provider-setup.md#-configuration-basics)
  * [Configuration Options Reference](provider-setup.md#configuration-options-reference)
* [☁️ Cloud Providers](provider-setup.md#️-cloud-providers)
  * [🟢 OpenAI (ChatGPT)](provider-setup.md#-openai-chatgpt)
  * [🟣 Claude (Anthropic)](provider-setup.md#-claude-anthropic)
  * [🔵 Gemini (Google)](provider-setup.md#-gemini-google)
  * [🔸 Grok (xAI)](provider-setup.md#-grok-xai)
  * [🤗 HuggingFace](provider-setup.md#-huggingface)
  * [⚡ Groq](provider-setup.md#-groq)
  * [🔷 DeepSeek](provider-setup.md#-deepseek)
  * [🟠 Mistral](provider-setup.md#-mistral)
  * [🌐 OpenRouter (Multi-Model Gateway)](provider-setup.md#-openrouter-multi-model-gateway)
  * [🔎 Perplexity](provider-setup.md#-perplexity)
  * [🧡 Cohere](provider-setup.md#-cohere)
  * [🚀 Voyage](provider-setup.md#-voyage)
  * [🟠 AWS Bedrock](provider-setup.md#-aws-bedrock)
  * [🐳 Docker Desktop AI Models](provider-setup.md#-docker-desktop-ai-models)
* [🦙 Local AI with Ollama](provider-setup.md#-local-ai-with-ollama)
  * [Why Ollama?](provider-setup.md#why-ollama)
  * [Installation Methods](provider-setup.md#installation-methods)
    * [Option 1: Native Installation](provider-setup.md#option-1-native-installation)
    * [Option 2: Docker (Recommended for Production)](provider-setup.md#option-2-docker-recommended-for-production)
  * [Pull and Configure Models](provider-setup.md#pull-and-configure-models)
  * [BoxLang Configuration](provider-setup.md#boxlang-configuration)
  * [Verify Installation](provider-setup.md#verify-installation)
  * [Model Selection Guide](provider-setup.md#model-selection-guide)
  * [Hardware Requirements](provider-setup.md#hardware-requirements)
* [🔐 Environment Variables](provider-setup.md#-environment-variables)
  * [In boxlang.json](provider-setup.md#in-boxlangjson)
  * [Set Environment Variables](provider-setup.md#set-environment-variables)
  * [Auto-Detection](provider-setup.md#auto-detection)
* [🔄 Multiple Providers](provider-setup.md#-multiple-providers)
  * [Provider Services](provider-setup.md#provider-services)
* [🔧 Troubleshooting](provider-setup.md#-troubleshooting)
  * [❌ "No API key provided"](provider-setup.md#-no-api-key-provided)
  * [⏱️ "Connection timeout"](provider-setup.md#️-connection-timeout)
  * [🔌 "Connection refused" (Ollama)](provider-setup.md#-connection-refused-ollama)
  * [🚫 "Model not found"](provider-setup.md#-model-not-found)
  * [💰 "Rate limit exceeded"](provider-setup.md#-rate-limit-exceeded)
  * [🔑 "Invalid API key"](provider-setup.md#-invalid-api-key)
* [🚀 Next Steps](provider-setup.md#-next-steps)
* [💡 Tips for Production](provider-setup.md#-tips-for-production)

***

## 🎯 Quick Provider Comparison

| Provider        | Type    | Best For                       | Cost   | Speed   | Context |
| --------------- | ------- | ------------------------------ | ------ | ------- | ------- |
| **OpenAI**      | Cloud   | General purpose, GPT-5         | \$$$   | Fast    | 128K    |
| **Claude**      | Cloud   | Long context, analysis         | \$$$   | Fast    | 200K    |
| **Gemini**      | Cloud   | Google integration, multimodal | \$$    | Fast    | 1M      |
| **Ollama**      | Local   | Privacy, offline, free         | Free   | Medium  | Varies  |
| **Groq**        | Cloud   | Ultra-fast inference           | \$$    | Fastest | 32K     |
| **DeepSeek**    | Cloud   | Code, reasoning                | $      | Fast    | 64K     |
| **HuggingFace** | Cloud   | Open-source models             | $      | Medium  | Varies  |
| **OpenRouter**  | Gateway | Access multiple models         | Varies | Fast    | Varies  |
| **Perplexity**  | Cloud   | Research, citations            | \$$    | Fast    | 8K      |
| **Cohere**      | Cloud   | Embeddings, multilingual       | \$$    | Fast    | 128K    |
| **Voyage**      | Cloud   | State-of-art embeddings        | \$$    | Fast    | N/A     |
| **Bedrock**     | Cloud   | AWS enterprise, multi-model    | \$$$   | Fast    | 200K    |
| **Docker Desktop** | Local   | Docker-managed models       | Free   | Fast    | Varies  |

### 💡 Recommendations by Use Case

* **General Chatbot**: OpenAI (GPT-4), Claude (Sonnet), Bedrock (Claude)
* **Long Documents**: Claude (200K context), Gemini (1M context), Bedrock (Claude)
* **Code Generation**: DeepSeek, OpenAI (GPT-4)
* **Fast Responses**: Groq, Gemini
* **Privacy/Offline**: Ollama (local), Docker Desktop (local)
* **Enterprise/AWS**: Bedrock (IAM auth, inference profiles)
* **Embeddings/RAG**: Voyage, Cohere, OpenAI
* **Research**: Perplexity (citations)
* **Cost-Effective**: Ollama (free), Docker Desktop (free), DeepSeek, Gemini
* **Multimodal**: Gemini, OpenAI (GPT-4), Bedrock (Claude)
* **Easy Local Setup**: Docker Desktop (managed), Ollama (native)

***

## 🔧 Configuration Basics

All providers are configured in your `boxlang.json` file:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openai",
        "apiKey": "your-api-key-here",
        "defaultParams": {
          "model": "gpt-4",
          "temperature": 0.7
        }
      }
    }
  }
}
```

### Configuration Options Reference

| Setting         | Type   | Default    | Description                     |
| --------------- | ------ | ---------- | ------------------------------- |
| `provider`      | string | `"openai"` | Default AI provider             |
| `apiKey`        | string | `""`       | API key for the provider        |
| `chatURL`       | string | Auto       | Custom API endpoint URL         |
| `defaultParams` | struct | `{}`       | Default parameters for requests |
| `timeout`       | number | `30`       | Request timeout in seconds      |
| `returnFormat`  | string | `"single"` | Default return format           |

***

## ☁️ Cloud Providers

### 🟢 OpenAI (ChatGPT)

**Best for**: General purpose AI, content generation, code assistance

**Get API Key**: [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openai",
        "apiKey": "sk-proj-...",
        "defaultParams": {
          "model": "gpt-4",
          "temperature": 0.7,
          "max_tokens": 2000
        }
      }
    }
  }
}
```

**Available Models**:

| Model           | Description           | Context | Best For                  |
| --------------- | --------------------- | ------- | ------------------------- |
| `gpt-5`         | Latest, most advanced | 128K    | Everything                |
| `gpt-4`         | Most capable          | 128K    | Complex tasks, reasoning  |
| `gpt-4-turbo`   | Faster, cheaper       | 128K    | Production apps           |
| `gpt-3.5-turbo` | Fast, affordable      | 16K     | Simple tasks, high volume |
| `gpt-4o`        | Optimized for chat    | 128K    | Conversational AI         |

**Pricing** (as of Dec 2024):

* GPT-5: \~$30/1M tokens input, \~$60/1M tokens output
* GPT-4: \~$10/1M tokens input, \~$30/1M tokens output
* GPT-4-Turbo: \~$5/1M tokens input, \~$15/1M tokens output
* GPT-3.5-Turbo: \~$0.50/1M tokens

**Usage Example**:

```javascript
// Use default configured provider
result = aiChat( "Explain quantum computing" )

// Override provider and model
result = aiChat(
    "Explain quantum computing",
    { model: "gpt-4-turbo" },
    { provider: "openai" }
)
```

***

### 🟣 Claude (Anthropic)

**Best for**: Long context analysis, detailed reasoning, safety-focused applications

**Get API Key**: [https://console.anthropic.com/](https://console.anthropic.com/)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "claude",
        "apiKey": "sk-ant-...",
        "defaultParams": {
          "model": "claude-3-5-sonnet-20241022",
          "max_tokens": 4096
        }
      }
    }
  }
}
```

**Available Models**:

| Model                        | Description            | Context | Best For         |
| ---------------------------- | ---------------------- | ------- | ---------------- |
| `claude-3-5-opus-20241022`   | Most capable           | 200K    | Complex analysis |
| `claude-3-5-sonnet-20241022` | Balanced (recommended) | 200K    | General use      |
| `claude-3-5-haiku-20241022`  | Fastest, cheapest      | 200K    | High volume      |

**Pricing**:

* Opus: \~$15/1M input, \~$75/1M output
* Sonnet: \~$3/1M input, \~$15/1M output
* Haiku: \~$0.25/1M input, \~$1.25/1M output

**Special Features**:

* **200K context window** - Entire books in one request
* **Constitutional AI** - Enhanced safety and helpfulness
* **Vision support** - Image analysis capabilities

***

### 🔵 Gemini (Google)

**Best for**: Google integration, multimodal content, massive context windows

**Get API Key**: [https://makersuite.google.com/app/apikey](https://makersuite.google.com/app/apikey)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "gemini",
        "apiKey": "AIza...",
        "defaultParams": {
          "model": "gemini-2.0-flash"
        }
      }
    }
  }
}
```

**Available Models**:

| Model              | Description                   | Context | Best For      |
| ------------------ | ----------------------------- | ------- | ------------- |
| `gemini-2.0-flash` | Fast, efficient (recommended) | 1M      | General use   |
| `gemini-1.5-pro`   | Most capable                  | 2M      | Complex tasks |
| `gemini-1.5-flash` | Fast, affordable              | 1M      | High volume   |

**Special Features**:

* **1-2M context window** - Process entire codebases
* **Multimodal native** - Text, images, audio, video
* **Free tier available** - Great for development

***

### 🔸 Grok (xAI)

**Best for**: Real-time data, Twitter/X integration, conversational AI

**Get API Key**: [https://console.x.ai/](https://console.x.ai/)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "grok",
        "apiKey": "xai-...",
        "defaultParams": {
          "model": "grok-2"
        }
      }
    }
  }
}
```

***

### 🤗 HuggingFace

**Best for**: Open-source models, community-driven, flexibility

**Get API Key**: [https://huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "huggingface",
        "apiKey": "hf_...",
        "defaultParams": {
          "model": "Qwen/Qwen2.5-72B-Instruct"
        }
      }
    }
  }
}
```

**Popular Models**:

* `Qwen/Qwen2.5-72B-Instruct` - Powerful general-purpose model
* `meta-llama/Llama-3.1-8B-Instruct` - Meta's Llama model
* `mistralai/Mistral-7B-Instruct-v0.3` - Fast and efficient

***

### ⚡ Groq

**Best for**: Ultra-fast inference with LPU architecture

**Get API Key**: [https://console.groq.com/](https://console.groq.com/)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "groq",
        "apiKey": "gsk_...",
        "defaultParams": {
          "model": "llama-3.1-70b-versatile"
        }
      }
    }
  }
}
```

**Special Features**:

* **Fastest inference** - Up to 500 tokens/second
* **LPU architecture** - Hardware-optimized for AI
* **Free tier** - Generous limits for testing

***

### 🔷 DeepSeek

**Best for**: Code generation, reasoning tasks, cost-effective

**Get API Key**: [https://platform.deepseek.com/](https://platform.deepseek.com/)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "deepseek",
        "apiKey": "sk-...",
        "defaultParams": {
          "model": "deepseek-chat"
        }
      }
    }
  }
}
```

***

### 🟠 Mistral

**Best for**: European data residency, balanced performance/cost

**Get API Key**: [https://console.mistral.ai/](https://console.mistral.ai/)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "mistral",
        "apiKey": "...",
        "defaultParams": {
          "model": "mistral-medium-latest"
        }
      }
    }
  }
}
```

**Available Models**:

* `mistral-large-latest` - Most capable
* `mistral-medium-latest` - Balanced
* `mistral-small-latest` - Fast, cost-effective

***

### 🌐 OpenRouter (Multi-Model Gateway)

**Best for**: Access multiple models through one API, cost optimization

**Get API Key**: [https://openrouter.ai/keys](https://openrouter.ai/keys)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "openrouter",
        "apiKey": "sk-or-...",
        "defaultParams": {
          "model": "openai/gpt-4"
        }
      }
    }
  }
}
```

**Special Features**:

* Access 100+ models through one API
* Automatic fallback if model unavailable
* Cost tracking across providers
* Free models available

***

### 🔎 Perplexity

**Best for**: Research, factual accuracy, citations

**Get API Key**: [https://www.perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "perplexity",
        "apiKey": "pplx-...",
        "defaultParams": {
          "model": "llama-3.1-sonar-large-128k-online"
        }
      }
    }
  }
}
```

**Special Features**:

* Real-time web search integration
* Automatic source citations
* Fact-checked responses

***

### 🧡 Cohere

**Best for**: Embeddings, multilingual support, RAG applications

**Get API Key**: [https://dashboard.cohere.com/api-keys](https://dashboard.cohere.com/api-keys)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "cohere",
        "apiKey": "...",
        "defaultParams": {
          "model": "command-r-plus"
        }
      }
    }
  }
}
```

**Special Features**:

* Best-in-class embeddings for RAG
* Native multilingual support (100+ languages)
* Tool use and structured output

***

### 🚀 Voyage

**Best for**: State-of-the-art embeddings optimized for RAG

**Get API Key**: [https://dash.voyageai.com/](https://dash.voyageai.com/)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "embeddingProvider": "voyage",
        "embeddingApiKey": "pa-...",
        "embeddingModel": "voyage-3"
      }
    }
  }
}
```

**Note**: Voyage is embedding-only (use with `aiEmbed()` or vector memory)

***

### 🟠 AWS Bedrock

**Best for**: Enterprise AWS deployments with Claude, Titan, Llama, and Mistral models

**Get Started**: [AWS Bedrock Console](https://console.aws.amazon.com/bedrock/)

**Configuration**:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "bedrock",
        "apiKey": "AWS_ACCESS_KEY_ID:AWS_SECRET_ACCESS_KEY",
        "defaultParams": {
          "model": "anthropic.claude-3-5-sonnet-20241022-v2:0"
        },
        "providerOptions": {
          "region": "us-east-1"
        }
      }
    }
  }
}
```

**Environment Variables**:

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_REGION="us-east-1"
```

**Example Usage**:

```javascript
// Basic chat
result = aiChat( provider: "bedrock", messages: "Hello!" )

// With inference profiles
result = aiChat(
    provider: "bedrock",
    messages: "Analyze this data",
    options: {
        providerOptions: {
            inferenceProfileArn: "arn:aws:bedrock:us-east-1:123456789012:inference-profile/my-profile"
        }
    }
)

// Streaming support
aiChatStream(
    provider: "bedrock",
    messages: "Tell me a story",
    onChunk: ( chunk ) => writeOutput( chunk.content )
)
```

**Available Models**:
- Claude: `anthropic.claude-3-5-sonnet-20241022-v2:0`, `anthropic.claude-3-opus-20240229`
- Titan: `amazon.titan-text-premier-v1:0`
- Llama: `meta.llama3-3-70b-instruct-v1:0`
- Mistral: `mistral.mistral-large-2407-v1:0`

**Special Features**:
* ✅ **Full streaming support** (v2.1.0+)
* ✅ **Inference profiles** for cost optimization
* ✅ **IAM-based authentication** for enterprise security
* ✅ **Cross-region inference** for global deployments
* ✅ **All model families**: Claude, Titan, Llama, Mistral

***

### 🐳 Docker Desktop AI Models

**Best for**: Local AI with Docker Desktop's built-in model support

**Get Started**: [Docker Desktop AI](https://docs.docker.com/desktop/features/ai/)

**Prerequisites**:
- Docker Desktop 4.36+ with AI features enabled
- Models available through Docker Desktop AI catalog

**Configuration**:

```javascript
// Docker Desktop models use OpenAI-compatible API
{
  "modules": {
    "bxai": {
      "settings": {
        "providers": {
          "docker-llama": {
            "params": {
              "model": "llama-3.2-1b"
            },
            "options": {
              "baseURL": "http://localhost:11435/v1"
            }
          },
          "docker-phi": {
            "params": {
              "model": "phi-4"
            },
            "options": {
              "baseURL": "http://localhost:11435/v1"
            }
          }
        }
      }
    }
  }
}
```

**Example Usage**:

```javascript
// Use Docker Desktop models
result = aiChat(
    provider: "docker-llama",
    messages: "What is BoxLang?"
)

// Stream responses
aiChatStream(
    provider: "docker-phi",
    messages: "Explain AI",
    onChunk: ( chunk ) => writeOutput( chunk.content )
)
```

**Available Models**:

* Meta Llama 3.2 (1B, 3B)
* Microsoft Phi-4
* Other models from Docker AI catalog

**Special Features**:

* ✅ **Zero configuration** - Models managed by Docker Desktop
* ✅ **No API keys required** - Local authentication
* ✅ **OpenAI-compatible** - Works with existing code
* ✅ **Easy model switching** - Managed through Docker Desktop UI
* ✅ **Privacy-first** - All processing local

***

## 🦙 Local AI with Ollama

**Perfect for privacy, offline use, and zero API costs!**

### Why Ollama?

* ✅ **100% Free** - No API costs ever
* ✅ **Privacy** - Data never leaves your machine
* ✅ **Offline** - Works without internet
* ✅ **No Rate Limits** - Use as much as you want
* ✅ **Fast** - Low latency on local hardware

### Installation Methods

#### Option 1: Native Installation

**macOS**:

```bash
brew install ollama
```

**Linux**:

```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

**Windows**: Download installer from [https://ollama.ai](https://ollama.ai)

#### Option 2: Docker (Recommended for Production)

See [Running Ollama with Docker](./#running-ollama-with-docker) in the installation guide.

### Pull and Configure Models

```bash
# Recommended general model
ollama pull llama3.2

# Smaller/faster options
ollama pull llama3.2:1b      # 1B parameters, very fast
ollama pull phi3             # Microsoft's efficient model

# Code-focused
ollama pull codellama
ollama pull deepseek-coder

# High quality
ollama pull mistral
ollama pull qwen2.5
```

### BoxLang Configuration

#### Basic Configuration

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

**Note**: Ollama doesn't require an API key for local use.

#### Custom Ollama Server URL (v2.1.0+)

If you're running Ollama on a custom port, remote server, or Docker container with custom configuration, use the predefined providers approach:

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "provider": "ollama",
        "providers": {
          "ollama": {
            "params": {
              "model": "llama3.2"
            },
            "options": {
              "baseURL": "http://my-ollama-server:11434"
            }
          },
          "ollama-docker": {
            "params": {
              "model": "qwen2.5:0.5b-instruct"
            },
            "options": {
              "baseURL": "http://192.168.1.100:11434"
            }
          }
        }
      }
    }
  }
}
```

Now you can use different Ollama instances:

```javascript
// Uses my-ollama-server:11434
result = aiChat( "Hello", { provider: "ollama" } )

// Uses Docker container at 192.168.1.100
result = aiChat( "Hello", { provider: "ollama-docker" } )
```

**💡 Benefits:**

- Configure once, use everywhere
- Easy to switch between local/remote Ollama instances
- No need to pass baseURL in every `aiModel()` or `aiChat()` call

**📖 See** [**Predefined Providers Configuration**](./#-predefined-providers-v210) for more details.

### Verify Installation

```bash
# Check Ollama is running
ollama list

# Test a model
ollama run llama3.2 "Hello!"

# Check from BoxLang
curl http://localhost:11434/api/tags
```

### Model Selection Guide

| Model         | Size | Speed | Quality | Use Case                  |
| ------------- | ---- | ----- | ------- | ------------------------- |
| `llama3.2:1b` | 1GB  | ⚡⚡⚡   | ⭐⭐      | Quick responses, testing  |
| `llama3.2`    | 3GB  | ⚡⚡    | ⭐⭐⭐     | General use (recommended) |
| `phi3`        | 2GB  | ⚡⚡⚡   | ⭐⭐⭐     | Balanced quality/speed    |
| `mistral`     | 4GB  | ⚡⚡    | ⭐⭐⭐⭐    | High quality responses    |
| `codellama`   | 4GB  | ⚡⚡    | ⭐⭐⭐⭐    | Code generation           |
| `qwen2.5:7b`  | 5GB  | ⚡     | ⭐⭐⭐⭐⭐   | Best quality (slower)     |

### Hardware Requirements

* **Minimum**: 8GB RAM, 4GB disk space
* **Recommended**: 16GB RAM, 10GB disk space
* **Optimal**: 32GB RAM, GPU (NVIDIA/AMD)

***

## 🔐 Environment Variables

Use environment variables to keep API keys out of config files:

### In boxlang.json

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

### Set Environment Variables

**macOS/Linux**:

```bash
export OPENAI_API_KEY="sk-..."
export CLAUDE_API_KEY="sk-ant-..."
export GEMINI_API_KEY="AIza..."
```

**Windows**:

```cmd
set OPENAI_API_KEY=sk-...
set CLAUDE_API_KEY=sk-ant-...
```

### Auto-Detection

**Convention:** By default, BoxLang AI automatically detects environment variables following the pattern `{PROVIDER_NAME}_API_KEY`. This means you don't need to explicitly configure API keys in `boxlang.json` if you set the appropriate environment variable.

For example:

* Setting `OPENAI_API_KEY` allows you to use OpenAI without configuration
* Setting `CLAUDE_API_KEY` allows you to use Claude without configuration
* And so on for all providers

**Automatically detected environment variables:**

* `OPENAI_API_KEY`
* `CLAUDE_API_KEY`
* `ANTHROPIC_API_KEY` (alternative for Claude)
* `GEMINI_API_KEY`
* `GOOGLE_API_KEY` (alternative for Gemini)
* `GROQ_API_KEY`
* `DEEPSEEK_API_KEY`
* `HUGGINGFACE_API_KEY`
* `HF_TOKEN` (alternative for HuggingFace)
* `MISTRAL_API_KEY`
* `PERPLEXITY_API_KEY`
* `COHERE_API_KEY`
* `VOYAGE_API_KEY`

***

## 🔄 Multiple Providers

Use different providers for different tasks:

```javascript
// Use OpenAI for general chat
chatResult = aiChat(
    "Explain AI",
    {},
    { provider: "openai", apiKey: getSystemSetting( "OPENAI_API_KEY" ) }
)

// Use Claude for deep analysis
analysisResult = aiChat(
    "Analyze this report: ${report}",
    { max_tokens: 8000 },
    { provider: "claude" }
)

// Use Ollama for privacy-sensitive data
privateResult = aiChat(
    "Process: ${sensitiveData}",
    {},
    { provider: "ollama" }
)

// Use Groq for fast responses
quickResult = aiChat(
    "Quick question",
    {},
    { provider: "groq" }
)
```

### Provider Services

Create reusable service instances:

```javascript
// Create services with default config
openaiService = aiService( "openai" )
claudeService = aiService( "claude" )
ollamaService = aiService( "ollama" )

// Or with custom configuration options (v2.1.0+)
openaiService = aiService( "openai", {
    apiKey: "sk-custom-key",
    timeout: 120,
    logResponse: true
})

ollamaService = aiService( "ollama", {
    baseURL: "http://my-ollama-server:11434"
})

// Use them in your application
generalChat = openaiService.invoke( request )
deepAnalysis = claudeService.invoke( complexRequest )
privateProcessing = ollamaService.invoke( sensitiveRequest )
```

***

## 🔧 Troubleshooting

### ❌ "No API key provided"

**Solution**: Set API key in config or pass directly

```javascript
// Option 1: Set in boxlang.json
{
  "modules": {
    "bxai": {
      "settings": {
        "apiKey": "your-key"
      }
    }
  }
}

// Option 2: Pass in request
answer = aiChat( "Hello", {}, { apiKey: "your-key" } )

// Option 3: Use environment variable
export OPENAI_API_KEY="sk-..."
```

### ⏱️ "Connection timeout"

**Solution**: Increase timeout setting

```json
{
  "modules": {
    "bxai": {
      "settings": {
        "timeout": 60
      }
    }
  }
}
```

### 🔌 "Connection refused" (Ollama)

**Check if Ollama is running**:

```bash
# Start Ollama
ollama serve

# Or check if running
curl http://localhost:11434/api/tags

# For Docker
docker compose -f docker-compose-ollama.yml ps
```

### 🚫 "Model not found"

**Solution**: Pull the model first (Ollama)

```bash
# List available models
ollama list

# Pull missing model
ollama pull llama3.2
```

### 💰 "Rate limit exceeded"

**Solutions**:

* Upgrade to paid tier
* Implement request caching
* Use different provider for high-volume tasks
* Switch to Ollama (no limits)

### 🔑 "Invalid API key"

**Verify**:

* Key is complete and not truncated
* Key is for correct provider
* Key has not expired
* Account has credits/subscription

***

## 🚀 Next Steps

* [**Installation Guide**](./) - Install the BoxLang AI module
* [**Quick Start**](../quickstart.md) - Your first AI conversation
* [**Basic Chatting**](../../main-components/chatting/basic-chatting.md) - Learn the fundamentals
* [**Advanced Features**](../../main-components/chatting/advanced-chatting.md) - Tools, streaming, multimodal

***

## 💡 Tips for Production

1. **Use environment variables** for API keys (never commit to git)
2. **Set appropriate timeouts** based on your use case
3. **Implement retry logic** for transient errors
4. **Monitor costs** with provider dashboards
5. **Use Ollama** for development/testing to save costs
6. **Cache responses** when possible to reduce API calls
7. **Choose right model** for each task (don't always use most expensive)
8. **Rate limit your application** to avoid provider rate limits
