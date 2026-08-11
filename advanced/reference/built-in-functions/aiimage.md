---
description: Generate images from prompts using supported AI providers.
icon: image
---

# aiImage

Generate images from prompts using supported AI providers.

## Syntax

```javascript
aiImage( prompt = "", params = {}, options = {} )
```

## Calling Modes

### Direct invocation

```javascript
response = aiImage( "A futuristic city skyline at sunset" )
```

### Fluent builder invocation

```javascript
response = aiImage()
    .prompt( "A watercolor fox in an autumn forest" )
    .provider( "openai" )
    .size( "1792x1024" )
    .high()
    .style( "vivid" )
    .generate()
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `prompt` | string | No | Text prompt. If omitted, returns fluent `AiImageRequest` builder |
| `params` | struct | No | Provider API parameters (`model`, `n`, `size`, `quality`, `style`, `format`) |
| `options` | struct | No | High-level options (`provider`, `apiKey`, `outputFile`, logging, timeout) |

## Common Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | module default | Provider name |
| `model` | string | provider default | Image model identifier |
| `size` | string | `auto` | Output size |
| `quality` | string | `auto` | Quality level |
| `style` | string | `""` | Style hint |
| `outputFile` | string | `""` | Save first image to file path |
| `outputFormat` | string | `url` | Provider payload format |
| `format` | string | `png` | Requested image format |
| `timeout` | numeric | `30` | Request timeout |

## Returns

| Condition | Return |
|---|---|
| Prompt provided and no `outputFile` | `AiImageResponse` |
| Prompt provided with `outputFile` | String file path |
| Prompt omitted | `AiImageRequest` fluent builder |

## AiImageResponse Methods

| Method | Description |
|---|---|
| `hasImages()` | True if response includes images |
| `getCount()` | Number of returned images |
| `getFirstImage()` | First image struct |
| `getFirstURL()` | URL for first image |
| `getFirstBase64()` | Base64 payload for first image |
| `getRevisedPrompt()` | Revised prompt when provider rewrites prompt |
| `getMimeType()` | MIME type from format |
| `toDataURI()` | Data URI for web embedding |
| `saveToFile( path )` | Save first image to disk |
| `saveAllToDirectory( path )` | Save all images |
| `toStruct()` | Metadata struct |

## Events Fired

| Event | When |
|---|---|
| `beforeAIImageGeneration` | Before image request is sent |
| `onAIImageRequest` | When image request payload is prepared |
| `onAIImageResponse` | After provider response received |
| `afterAIImageGeneration` | After final response processing |

## Examples

### Direct generation

```javascript
img = aiImage( "A cinematic neon city at night" )
img.saveToFile( expandPath( "/tmp/city.png" ) )
```

### Save directly during call

```javascript
filePath = aiImage(
    "A minimal logo icon of a robot",
    {},
    { outputFile: expandPath( "/tmp/logo.png" ) }
)
```

### Fluent with shortcuts

```javascript
result = aiImage()
    .prompt( "A photoreal mountain landscape" )
    .provider( "gemini" )
    .landscape()
    .high()
    .asWebp()
    .generate()
```
