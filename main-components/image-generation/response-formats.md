---
description: >-
  The AiImageResponse object wraps generated images with methods for saving,
  encoding, embedding, and inspecting metadata.
icon: file-image
---

# Image Response & Formats

When `aiImage()` completes successfully, it returns an `AiImageResponse` object containing one or more generated images with convenience methods for saving, encoding, and inspection.

## 📦 `AiImageResponse` Methods

| Method | Returns | Description |
|---|---|---|---|
| `hasImages()` | boolean | `true` if images are present and non-empty |
| `getCount()` | numeric | Number of generated images |
| `getFirstImage()` | struct | First image struct `{url, data, mimeType, revisedPrompt}` (or empty struct) |
| `getFirstURL()` | string | URL of the first image (if provider returns URLs) |
| `getFirstBase64()` | string | Base64-encoded data of the first image (fetches URL if needed) |
| `getRevisedPrompt()` | string | Provider's revised prompt (if any) |
| `saveToFile( path )` | string | Save first image to file; returns absolute path |
| `saveAllToDirectory( dir )` | array | Save all images to directory; returns array of paths |
| `toDataURI()` | string | `data:image/png;base64,...` URI for HTML `<img>` `src` |
| `getMimeType()` | string | MIME type (e.g. `image/png`, `image/jpeg`) |
| `toStruct()` | struct | Metadata struct (no binary data — safe for logging) |
| `toJSON()` | string | JSON-serialized metadata |
| `getMetadataValue( key )` | any | Read a value from the response metadata bag |
| `setMetadataValue( key, value )` | this | Write a value to the metadata bag (fluent, chainable) |

## 💾 Saving Images

### Save Single Image

```javascript
response = aiImage( "A sunset over the ocean" )
path = response.saveToFile( "/images/sunset.png" )
println( "Saved to: #path#" )
```

### Save All Images

```javascript
response = aiImage( "Four seasons landscape", { n: 4 } )
paths = response.saveAllToDirectory( "/images/seasons/" )

// paths = [
//   "/images/seasons/image_1.png",
//   "/images/seasons/image_2.png",
//   "/images/seasons/image_3.png",
//   "/images/seasons/image_4.png"
// ]
```

## 🌐 Embedding in HTML

### Data URI

```javascript
response = aiImage( "A cute cartoon cat" )
dataURI = response.toDataURI()

// Ready for HTML
html = '<img src="#dataURI#" alt="AI generated cat" />'
```

### Base64 for APIs

```javascript
response = aiImage( "Product photo" )
base64 = response.getFirstBase64()
mimeType = response.getMimeType()

// Send to frontend or store in database
apiResponse = {
    image: base64,
    mimeType: mimeType,
    revisedPrompt: response.getRevisedPrompt()
}
```

## 🔍 Inspecting Responses

```javascript
response = aiImage( "A mountain landscape" )

// Check if images were generated
if ( response.hasImages() ) {
    println( "Generated #response.getCount()# image(s)" )
    println( "MIME type: #response.getMimeType()#" )
    println( "Revised prompt: #response.getRevisedPrompt()#" )

    // Inspect the first image struct
    firstImage = response.getFirstImage()
    println( "First image URL: #firstImage.url ?: '(none)'#" )
    println( "First image revised prompt: #firstImage.revisedPrompt ?: '(none)'#" )
}

// Get metadata struct (safe for logging — no binary data)
metadata = response.toStruct()
```

## 📋 `AiImageRequest` Properties

The request object carries all generation parameters:

| Property | Type | Default | Description |
|---|---|---|---|
| `prompt` | string | `""` | Text description of the image |
| `n` | numeric | `1` | Number of images to generate |
| `size` | string | `auto` | Image dimensions (e.g. `"1024x1024"`, `"1792x1024"`) |
| `quality` | string | `auto` | Quality level (`low`, `medium`, `high`, `auto`) |
| `style` | string | `""` | Visual style (`vivid`, `natural`) |
| `instructions` | string | `""` | Additional style / tone / mood instructions appended to the prompt |
| `format` | string | `png` | Output image format: `png`, `jpeg`, `webp` |
| `outputFormat` | string | `url` | How the provider returns data (`url` or `b64_json`) |
| `outputFile` | string | `""` | File path for direct saving |

All properties are accessible via BoxLang property conventions (auto-generated getters/setters).

### Fluent Builder Properties

When using the fluent builder API, `AiImageRequest` also exposes convenience size and quality aliases:

```javascript
// Size aliases
.square()          // 1024x1024
.landscape()       // 1536x1024
.portrait()        // 1024x1536
.twoKSquare()      // 2048x2048
.twoKLandscape()   // 2048x1152
.fourKLandscape()  // 3840x2160
.fourKPortrait()   // 2160x3840

// Quality helpers
.low()
.medium()
.high()
.auto()

// Format helpers
.asPng()
.asJpeg()
.asWebp()
```
