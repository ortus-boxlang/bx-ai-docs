---
description: Events fired around AI image generation requests and responses.
icon: image
---

# Image Events

### beforeAIImageGeneration

Fired before an image generation request is sent to the provider.

**When**: Before `aiImage()` sends the HTTP request

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `imageRequest` | `AiImageRequest` | The image generation request object |
| `service` | `IAiImageService` | The image service instance |

#### Example

```javascript
BoxRegisterInterceptor( "beforeAIImageGeneration", function( event ) {
    println( "Generating image: prompt='#event.imageRequest.prompt#'" )
})
```

***

### afterAIImageGeneration

Fired after an image generation response is received from the provider.

**When**: After `aiImage()` receives the response

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `imageRequest` | `AiImageRequest` | The image generation request object |
| `service` | `IAiImageService` | The image service instance |
| `result` | `AiImageResponse` | The generated image response |

#### Example

```javascript
BoxRegisterInterceptor( "afterAIImageGeneration", function( event ) {
    println( "Generated #event.result.getCount()# image(s) via #event.service.getName()#" )
})
```

***

### onAIImageRequest

Fired when an image request object is created.

**When**: During `aiImage()` request construction

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `imageRequest` | `AiImageRequest` | The image request object |

#### Example

```javascript
BoxRegisterInterceptor( "onAIImageRequest", function( event ) {
    // Modify request parameters
    event.imageRequest.setSize( "1792x1024" )
})
```

***

### onAIImageResponse

Fired after an image response is received from the provider.

**When**: After the provider returns generated images

#### Event Arguments

| Argument | Type | Description |
|---|---|---|
| `response` | `AiImageResponse` | The image response |
| `rawResponse` | struct | Raw provider response |
| `provider` | string | Provider name |

#### Example

```javascript
BoxRegisterInterceptor( "onAIImageResponse", function( event ) {
    logMetric( "ai.image.generated", {
        provider: event.provider,
        count: event.result.getCount()
    })
})
```
