---
description: >-
  Generate images from text prompts using aiImage(). Learn about parameters,
  options, provider-specific settings, and configuration.
icon: wand-magic-sparkles
---

# Generating Images

The `aiImage()` BIF generates one or more images from a text description. It works across all providers that implement `IAiImageService`.

## 🔧 The `aiImage()` Function

### Syntax

```javascript
aiImage( prompt, params={}, options={} )
```

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `prompt` | string | ✅ Yes | Text description of the image to generate |
| `params` | struct | No | Provider API parameters |
| `options` | struct | No | Module-level options |

### `params` — Provider Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `n` | numeric | `1` | Number of images to generate |
| `size` | string | `1024x1024` | Image dimensions (OpenAI) or aspect ratio (Gemini) |
| `quality` | string | `standard` | Image quality: `standard`, `hd` (OpenAI only) |
| `style` | string | `vivid` | Image style: `vivid`, `natural` (OpenAI only) |

### `options` — Module Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config) | AI provider: `openai`, `gemini`, `grok`, `openrouter` |
| `apiKey` | string | (env var) | Provider API key |
| `outputFile` | string | `""` | If non-empty, saves the first image to this path and returns the file path string |
| `timeout` | numeric | `30` | HTTP request timeout in seconds |
| `logRequest` | boolean | `false` | Log requests to the module log file |
| `logRequestToConsole` | boolean | `false` | Print request payload to console |
| `logResponse` | boolean | `false` | Log responses to the module log file |
| `logResponseToConsole` | boolean | `false` | Print raw provider response to console |

## 🎨 Provider-Specific Settings

### OpenAI

```javascript
// OpenAI with full controls
response = aiImage(
    "A photorealistic portrait of a cat wearing a crown",
    {
        n: 4,
        size: "1024x1024",
        quality: "hd",
        style: "natural"
    },
    { provider: "openai" }
)
```

**Supported sizes:** `256x256`, `512x512`, `1024x1024`, `1024x1792`, `1792x1024`

**Quality options:** `standard`, `hd`

**Style options:** `vivid`, `natural`

### Gemini (Imagen 3)

```javascript
// Gemini — size maps to aspect ratio
response = aiImage(
    "A serene Japanese garden with cherry blossoms",
    { size: "16:9" },
    { provider: "gemini" }
)
```

**Supported aspect ratios:** `1:1`, `16:9`, `9:16`

### Grok / xAI

```javascript
// Grok image generation
response = aiImage(
    "A cyberpunk city at night with neon lights",
    { size: "1024x1024" },
    { provider: "grok" }
)
```

### OpenRouter

```javascript
// OpenRouter — FLUX Schnell by default
response = aiImage(
    "An oil painting of a lighthouse on a stormy coast",
    { size: "1024x1024" },
    { provider: "openrouter" }
)
```

## 💡 Examples

### Generate and Save to File

```javascript
// Using outputFile option — returns file path string
path = aiImage(
    "A minimalist logo design for a tech startup",
    {},
    { outputFile: "/images/logo.png" }
)
println( "Saved to: #path#" )
```

### Generate Multiple Images

```javascript
response = aiImage(
    "A set of abstract geometric patterns",
    { n: 4, size: "512x512" }
)

// Save all to a directory
paths = response.saveAllToDirectory( "/images/patterns/" )
println( "Saved #paths.len()# images" )
```

### Get Image as Base64

```javascript
response = aiImage( "A simple icon" )
base64 = response.getFirstBase64()

// Use in API response or storage
apiResponse = { image: base64, mimeType: response.getMimeType() }
```

### Access Revised Prompt

Some providers (like OpenAI) may revise your prompt for better results:

```javascript
response = aiImage( "A dog" )
println( "Original: A dog" )
println( "Revised: #response.getRevisedPrompt()#" )
// "A golden retriever sitting in a sunny meadow, photorealistic style"
```
