---
description: >-
  Generate images from text prompts using aiImage(). Learn about parameters,
  options, provider-specific settings, and the fluent builder API.
icon: wand-magic-sparkles
---

# Generating Images

The `aiImage()` BIF generates one or more images from a text description. It works across all providers that implement `IAiImageService`.

The BIF supports **two calling conventions**:

1. **Direct call** — Pass a prompt string and receive an `AiImageResponse` (or file path)
2. **Fluent builder** — Call `aiImage()` with no arguments (or use `AiImageRequest.of()`), chain expressive methods, then call `.generate()` as the terminator

## 🔧 The `aiImage()` Function

### Syntax

```javascript
// Direct call — returns AiImageResponse
aiImage( prompt, params={}, options={} )

// Fluent builder — call with no prompt, chain methods, terminate with .generate()
aiImage()
    .prompt( "..." )
    .provider( "openai" )
    .landscape()
    .high()
    .generate()

// Static factory — same as aiImage() with no prompt
AiImageRequest.of( "..." )
    .provider( "gemini" )
    .generate()
```

### Direct Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `prompt` | string | ✅ Yes | Text description of the image to generate |
| `params` | struct | No | Provider API parameters |
| `options` | struct | No | Module-level options |

### `params` — Provider Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `n` | numeric | `1` | Number of images to generate |
| `size` | string | `auto` | Image dimensions, e.g. `"1024x1024"`, `"1792x1024"`, `"1024x1792"` |
| `quality` | string | `auto` | Image quality: `low`, `medium`, `high`, `xhigh`, `max`, `auto` (OpenAI GPT Image models) |
| `style` | string | `""` | Visual style, provider-specific |
| `instructions` | string | `""` | Additional style / tone / mood instructions appended to the prompt |
| `outputFormat` | string | `url` | How the provider returns image data: `url` or `b64_json` |
| `format` | string | `png` | Output image format: `png`, `jpeg`, or `webp` (provider-specific) |
| `outputCompression` | numeric | — | Compression level for OpenAI GPT Image models (`output_compression`, jpeg and webp) |

### `options` — Module Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config) | AI provider: `openai`, `gemini`, `grok`, `openrouter` |
| `apiKey` | string | (env var) | Provider API key |
| `model` | string | (provider default) | Model override (e.g. `"gpt-image-2.5-sunburst"`) |
| `n` | numeric | `1` | Number of images to generate |
| `size` | string | `auto` | Image dimensions |
| `quality` | string | `auto` | Image quality level |
| `style` | string | `""` | Visual style |
| `instructions` | string | `""` | Additional instructions appended to prompt |
| `outputFormat` | string | `url` | Response format: `url` or `b64_json` |
| `format` | string | `png` | Output image format: `png`, `jpeg`, `webp` |
| `outputCompression` | numeric | — | Compression level (OpenAI GPT Image models) |
| `outputFile` | string | `""` | If set, saves the first image to this path and returns the file path string |
| `timeout` | numeric | `30` | HTTP request timeout in seconds |
| `logRequest` | boolean | `false` | Log requests to the module log file |
| `logRequestToConsole` | boolean | `false` | Print request payload to console |
| `logResponse` | boolean | `false` | Log responses to the module log file |
| `logResponseToConsole` | boolean | `false` | Print raw provider response to console |

## 🧩 Fluent Builder API

When `aiImage()` is called **with no prompt** (or via `AiImageRequest.of()`), it returns an `AiImageRequest` instance that acts as a fluent builder. Chain expressive methods and terminate with `.generate()`.

### Builder Methods

| Method | Description |
|---|---|
| `.prompt( text )` | Set the text description of the image |
| `.model( name )` | Set the AI model (e.g. `"gpt-image-2.5-flare"`, `"gpt-image-2.5-sunburst"`) |
| `.provider( name )` | Set the provider (`"openai"`, `"gemini"`, `"grok"`, `"openrouter"`) |
| `.apiKey( key )` | Set the API key |
| `.size( dims )` | Set image dimensions, e.g. `"1024x1024"`, `"1792x1024"`, `"1024x1792"` |
| `.quality( level )` | Set quality: `"low"`, `"medium"`, `"high"`, `"auto"` |
| `.style( name )` | Set visual style (`"vivid"`, `"natural"`) |
| `.instructions( text )` | Set additional style / tone / mood instructions |
| `.n( count )` | Set number of images to generate |
| `.format( fmt )` | Set output image format: `"png"`, `"jpeg"`, `"webp"` |
| `.outputFormat( fmt )` | Set provider response format: `"url"` or `"b64_json"` |
| `.outputFile( path )` | Automatically save the first image to this file path on `.generate()` |
| `.outputCompression( value )` | Set compression level (OpenAI GPT Image models) |
| `.withOptions( struct )` | Apply a struct of high-level options in bulk |
| `.withParams( struct )` | Merge additional provider API parameters |
| `.withLogging( request, response, console )` | Enable request/response logging |

#### Size Convenience Methods

| Method | Resolution |
|---|---|
| `.square()` | `1024x1024` |
| `.landscape()` | `1536x1024` |
| `.portrait()` | `1024x1536` |
| `.twoKSquare()` | `2048x2048` |
| `.twoKLandscape()` | `2048x1152` |
| `.fourKLandscape()` | `3840x2160` |
| `.fourKPortrait()` | `2160x3840` |

#### Quality Convenience Methods

| Method | Effect |
|---|---|
| `.low()` | Sets quality to `"low"` |
| `.medium()` | Sets quality to `"medium"` |
| `.high()` | Sets quality to `"high"` |
| `.auto()` | Sets quality to `"auto"` |

#### Format Convenience Methods

| Method | Effect |
|---|---|
| `.asPng()` | Sets output format to `"png"` |
| `.asJpeg()` | Sets output format to `"jpeg"` |
| `.asWebp()` | Sets output format to `"webp"` |

### Fluent Builder Examples

```javascript
// Basic fluent chain
response = aiImage()
    .prompt( "a breathtaking mountain sunrise" )
    .provider( "openai" )
    .size( "1536x1024" )
    .high()
    .generate()

// Using size aliases and quality helpers
response = aiImage()
    .prompt( "a cozy coffee shop interior, warm lighting" )
    .model( "gpt-image-2.5-sunburst" )
    .instructions( "painterly style, impressionist brush strokes" )
    .outputFile( "/tmp/coffeeshop.png" )
    .generate()

// Using AiImageRequest.of() static factory
response = AiImageRequest
    .of( "a red fox in an autumn forest" )
    .provider( "gemini" )
    .landscape()
    .generate()

// Format conversion
response = aiImage()
    .prompt( "a minimalist logo, flat design" )
    .asWebp()
    .generate()

// Gemini with size aliases
response = aiImage()
    .prompt( "a fox in an autumn forest" )
    .provider( "gemini" )
    .landscape()
    .generate()

// Output compression (OpenAI GPT Image models)
path = aiImage()
    .prompt( "a watercolor painting of a mountain lake" )
    .outputCompression( 80 )
    .outputFile( "/images/mountain.webp" )
    .generate()
// Returns the file path string
```

### `generate()` Terminator

The `.generate()` method executes image generation with all the options set in the chain. It returns:

- **`AiImageResponse`** — When no `outputFile` is set
- **String (file path)** — When `outputFile` is set

Internally, `.generate()` calls `aiImage()` with the accumulated state, which fires all events and validates provider capabilities.

## 🎨 Provider-Specific Settings

### OpenAI

OpenAI image generation uses the GPT Image models:

- **`gpt-image-2.5-flare`** (default) — Fast, high-quality everyday image generation
- **`gpt-image-2.5-sunburst`** — OpenAI's most capable model for image generation and editing

`gpt-image-1` was shut down on 2026-10-23 and DALL-E 2 and 3 on 2026-05-12. See OpenAI's [deprecations](https://developers.openai.com/api/docs/deprecations).

```javascript
// OpenAI with the default model — format and compression
response = aiImage(
    "A photorealistic portrait of a cat wearing a crown",
    { format: "webp" },
    { provider: "openai" }
)

// The most capable model, with quality and size
response = aiImage(
    "A photorealistic portrait of a cat wearing a crown",
    {
        model: "gpt-image-2.5-sunburst",
        size: "1024x1024",
        quality: "high"
    },
    { provider: "openai" }
)

// Fluent builder with format and compression
response = aiImage()
    .prompt( "A watercolor painting of a mountain lake" )
    .provider( "openai" )
    .landscape()
    .asWebp()
    .outputCompression( 85 )
    .generate()
```

**Sizes:** `1024x1024`, `1536x1024`, `1024x1536`, or a custom `WIDTHxHEIGHT` (multiples of 16, aspect ratio between 1:3 and 3:1)

**Quality options:** `low`, `medium`, `high`, `xhigh`, `max`, `auto` (default)

**Format options:** `png` (default), `jpeg`, `webp`. `outputCompression` (0 to 100) applies to `jpeg` and `webp`.

### Gemini (Imagen)

```javascript
// Gemini — standard size strings are mapped to aspect ratios
response = aiImage(
    "A serene Japanese garden with cherry blossoms",
    { size: "1792x1024" },
    { provider: "gemini" }
)

// Fluent builder with Gemini
response = aiImage()
    .prompt( "a fox in an autumn forest" )
    .provider( "gemini" )
    .landscape()
    .generate()
```

Gemini maps common size strings to aspect ratios:

| Size | Aspect Ratio |
|---|---|
| `1024x1024`, `512x512`, `256x256` | `1:1` |
| `1792x1024` | `16:9` |
| `1024x1792` | `9:16` |

### Grok / xAI

```javascript
// Grok image generation (OpenAI-compatible)
response = aiImage(
    "A cyberpunk city at night with neon lights",
    { size: "1024x1024" },
    { provider: "grok" }
)

// Fluent builder
response = aiImage()
    .prompt( "A cyberpunk city at night with neon lights" )
    .provider( "grok" )
    .square()
    .generate()
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

### Direct Call — Generate and Save to File

```javascript
// Using outputFile option — returns file path string
path = aiImage(
    "A minimalist logo design for a tech startup",
    {},
    { outputFile: "/images/logo.png" }
)
println( "Saved to: #path#" )
```

### Fluent Builder — Generate and Save to File

```javascript
// Fluent builder with outputFile — also returns file path string
path = aiImage()
    .prompt( "A minimalist logo design for a tech startup" )
    .provider( "openai" )
    .square()
    .outputFile( "/images/logo.png" )
    .generate()
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

### Fluent Builder — WebP with Compression

```javascript
// Use the GPT Image format and compression features via fluent builder
path = aiImage()
    .prompt( "A watercolor painting of a mountain lake" )
    .provider( "openai" )
    .landscape()
    .asWebp()
    .outputCompression( 80 )
    .outputFile( "/images/mountain.webp" )
    .generate()
println( "Saved compressed image to: #path#" )
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

### Using AiImageRequest.of() Static Factory

```javascript
// The static factory is equivalent to aiImage() with no prompt
response = AiImageRequest
    .of( "a red fox in an autumn forest" )
    .provider( "gemini" )
    .landscape()
    .generate()
println( "Generated #response.getCount()# image(s)" )
```
