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
| `n` | numeric | `1` | Number of images to generate (DALL-E 3 max is 1) |
| `size` | string | `auto` | Image dimensions, e.g. `"1024x1024"`, `"1792x1024"`, `"1024x1792"` |
| `quality` | string | `auto` | Image quality: `standard`, `hd` (OpenAI DALL-E 3); `low`, `medium`, `high` (gpt-image-1) |
| `style` | string | `""` | Visual style: `vivid`, `natural` (OpenAI DALL-E 3); provider-specific elsewhere |
| `instructions` | string | `""` | Additional style / tone / mood instructions appended to the prompt |
| `outputFormat` | string | `url` | How the provider returns image data: `url` or `b64_json` |
| `format` | string | `png` | Output image format: `png`, `jpeg`, or `webp` (provider-specific) |
| `outputCompression` | numeric | — | Compression level for OpenAI `gpt-image-1` (`output_compression`) |

### `options` — Module Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config) | AI provider: `openai`, `gemini`, `grok`, `openrouter` |
| `apiKey` | string | (env var) | Provider API key |
| `model` | string | (provider default) | Model override (e.g. `"dall-e-3"`, `"gpt-image-1"`) |
| `n` | numeric | `1` | Number of images to generate |
| `size` | string | `auto` | Image dimensions |
| `quality` | string | `auto` | Image quality level |
| `style` | string | `""` | Visual style |
| `instructions` | string | `""` | Additional instructions appended to prompt |
| `outputFormat` | string | `url` | Response format: `url` or `b64_json` |
| `format` | string | `png` | Output image format: `png`, `jpeg`, `webp` |
| `outputCompression` | numeric | — | Compression level (OpenAI `gpt-image-1`) |
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
| `.model( name )` | Set the AI model (e.g. `"gpt-image-1"`, `"dall-e-3"`) |
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
| `.outputCompression( value )` | Set compression level (OpenAI `gpt-image-1`) |
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
    .size( "1792x1024" )
    .high()
    .style( "vivid" )
    .generate()

// Using size aliases and quality helpers
response = aiImage()
    .prompt( "a cozy coffee shop interior, warm lighting" )
    .model( "dall-e-3" )
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

// Output compression (OpenAI gpt-image-1)
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

OpenAI offers two model families for image generation:

- **`gpt-image-1`** (default) — Latest model with `format` (png/jpeg/webp) and `outputCompression` support
- **`dall-e-3`** — Earlier model with `quality` and `style` controls

```javascript
// OpenAI with gpt-image-1 (default) — format and compression
response = aiImage(
    "A photorealistic portrait of a cat wearing a crown",
    { format: "webp" },
    { provider: "openai" }
)

// OpenAI DALL-E 3 with quality and style
response = aiImage(
    "A photorealistic portrait of a cat wearing a crown",
    {
        n: 1,
        size: "1024x1024",
        quality: "hd",
        style: "natural"
    },
    { provider: "openai" }
)

// Fluent builder with gpt-image-1 features
response = aiImage()
    .prompt( "A watercolor painting of a mountain lake" )
    .provider( "openai" )
    .landscape()
    .asWebp()
    .outputCompression( 85 )
    .generate()
```

**Supported sizes (DALL-E 3):** `1024x1024`, `1024x1792`, `1792x1024`

**Quality options (DALL-E 3):** `standard`, `hd`

**Style options (DALL-E 3):** `vivid`, `natural`

**Format options (gpt-image-1):** `png`, `jpeg`, `webp`

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
// Use gpt-image-1 format and compression features via fluent builder
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
