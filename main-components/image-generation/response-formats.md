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
|---|---|---|
| `hasImages()` | boolean | `true` if images are present and non-empty |
| `getCount()` | numeric | Number of generated images |
| `getFirstURL()` | string | URL of the first image (if provider returns URLs) |
| `getFirstBase64()` | string | Base64-encoded data of the first image |
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
}

// Get metadata struct (safe for logging — no binary data)
metadata = response.toStruct()
```

## 📋 `AiImageRequest` Properties

The request object carries all generation parameters:

| Property | Type | Description |
|---|---|---|
| `prompt` | string | Text description of the image |
| `n` | numeric | Number of images to generate |
| `size` | string | Image dimensions or aspect ratio |
| `quality` | string | Quality level (`standard`, `hd`) |
| `style` | string | Visual style (`vivid`, `natural`) |
| `instructions` | string | Additional generation instructions |
| `outputFormat` | string | Output format preference |
| `outputFile` | string | File path for direct saving |

All properties are accessible via BoxLang property conventions (auto-generated getters/setters).
