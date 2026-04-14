---
description: >-
  Convert any text to natural-sounding audio using AI providers. aiSpeak()
  returns binary audio data wrapped in an AiSpeechResponse with helpers for
  saving, encoding, and streaming.
icon: microphone
---

# 🎤 Text-to-Speech

`aiSpeak()` converts text to natural-sounding speech using cloud AI providers. It returns an `AiSpeechResponse` object containing the binary audio data, with convenience methods for saving to disk, encoding as Base64, generating data URIs for HTML playback, and inspecting metadata.

## 🔧 The `aiSpeak()` Function

### Syntax

```javascript
aiSpeak( text, params={}, options={} )
```

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `text` | string | ✅ Yes | The text to synthesize into speech |
| `params` | struct | No | Provider API parameters such as `model`, `voice`, `speed` |
| `options` | struct | No | Module-level options such as `provider`, `apiKey`, `outputFile` |

### Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config) | AI provider: `openai`, `mistral`, `gemini`, `grok`, `elevenlabs` |
| `apiKey` | string | (env var) | Provider API key (falls back to `<PROVIDER>_API_KEY` env var) |
| `voice` | string | (config) | Voice name or ID for the provider |
| `outputFormat` | string | `mp3` | Audio output format: `mp3`, `wav`, `flac`, `opus`, `pcm` |
| `speed` | numeric | `1.0` | Playback speed multiplier (range: 0.25–4.0) |
| `outputFile` | string | `""` | If non-empty, saves audio to this path and returns the file path string instead of `AiSpeechResponse` |
| `timeout` | numeric | `30` | HTTP request timeout in seconds |
| `logRequest` | boolean | `false` | Log requests to the module log file |
| `logRequestToConsole` | boolean | `false` | Print request payload to console |
| `logResponse` | boolean | `false` | Log responses to the module log file |
| `logResponseToConsole` | boolean | `false` | Print raw provider response to console |

## 📦 Return Value — `AiSpeechResponse`

When `outputFile` is **not** set, `aiSpeak()` returns an `AiSpeechResponse` object wrapping the binary audio data.

When `outputFile` **is** set, `aiSpeak()` saves the audio to that path and returns the **absolute file path string** instead of the response object.

### `AiSpeechResponse` Methods

| Method | Returns | Description |
|---|---|---|
| `saveToFile( filePath )` | string | Saves audio binary to disk; returns the absolute path |
| `getBase64()` | string | Returns the audio data as a Base64-encoded string |
| `getMimeType()` | string | Returns the MIME type (e.g. `audio/mpeg` for `mp3`) |
| `toDataURI()` | string | Returns a `data:audio/mpeg;base64,...` URI for an HTML `<audio>` `src` attribute |
| `hasAudio()` | boolean | Returns `true` if audio binary data is present and non-empty |
| `getSize()` | numeric | Returns the size of the audio data in bytes |
| `getAudioFormat()` | string | Returns the audio format string (e.g. `mp3`, `wav`) |
| `toStruct()` | struct | Returns a metadata struct (no binary data — safe for logging) |
| `toJSON()` | string | Returns JSON-serialized metadata |
| `getMetadataValue( key )` | any | Read a value from the response metadata bag |
| `setMetadataValue( key, value )` | this | Write a value to the metadata bag (fluent, chainable) |

## 💡 Examples

### Basic — synthesize and save to file

```javascript
audio = aiSpeak( "Welcome to BoxLang AI!" )
audio.saveToFile( "welcome.mp3" )
println( "Size: #audio.getSize()# bytes" )
```

### Shorthand with `outputFile` option

When you only need the file on disk, use `outputFile` to skip the response object entirely:

```javascript
path = aiSpeak(
    "Your order has shipped.",
    {},
    { outputFile: "/audio/notification.mp3" }
)
println( "Saved to: #path#" )
```

### Custom provider and voice

```javascript
audio = aiSpeak(
    "Hello, this is a custom voice.",
    { speed: 1.2 },
    { provider: "openai", voice: "nova", outputFormat: "wav" }
)
audio.saveToFile( "custom.wav" )
```

### Base64 / data URI for web responses

Embed audio directly in an HTML page or API response — no file I/O required:

```javascript
audio = aiSpeak( "Click to play this message" )

// Embed directly in HTML
htmlOutput = '<audio controls src="#audio.toDataURI()#"></audio>'

// Or pass the raw Base64 to a front-end
jsonResponse = { audio: audio.getBase64(), mimeType: audio.getMimeType() }
```

### ElevenLabs — high-quality multilingual voice

```javascript
audio = aiSpeak(
    "This is a premium voice synthesis example.",
    { voice_id: "21m00Tcm4TlvDq8ikWAM" },  // Rachel voice ID
    { provider: "elevenlabs" }
)
audio.saveToFile( "premium.mp3" )
println( "Format: #audio.getAudioFormat()#, Size: #audio.getSize()# bytes" )
```

### Generate comparison files across all voices

```javascript
voices = [ "alloy", "echo", "fable", "onyx", "nova", "shimmer" ]
sampleText = "BoxLang AI — where imagination meets voice."

voices.each( function( voice ) {
    aiSpeak(
        sampleText,
        { voice: voice },
        { outputFile: expandPath( "/tmp/voice-#voice#.mp3" ) }
    )
    println( "Generated: /tmp/voice-#voice#.mp3" )
})
```

## 🎙️ Provider Voice Reference

| Provider | Available Voices | Default |
|---|---|---|
| **OpenAI** | `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer` | `alloy` |
| **Mistral** | `Charlotte` | `Charlotte` |
| **Gemini** | `Kore` (and others via API) | `Kore` |
| **Grok / xAI** | `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer`, `eve` | provider default |
| **ElevenLabs** | Voice IDs from your ElevenLabs voice library | provider default |

> For ElevenLabs, pass a `voice_id` in `params` rather than a voice name. Find voice IDs in your ElevenLabs dashboard or via the ElevenLabs API.

## 📡 Events

`aiSpeak()` fires two interception points you can hook into for logging, auditing, or modifying behavior.

| Event | Data Available |
|---|---|
| `beforeAISpeech` | `speechRequest`, `service` |
| `afterAISpeech` | `speechRequest`, `service`, `result` |

```javascript
// Log all TTS calls with their output size
BoxRegisterInterceptor( "afterAISpeech", function( event ) {
    var sizeKB = event.result.getSize() / 1024
    println( "TTS: provider=#event.service.getName()# size=#numberFormat( sizeKB, '0.0' )#KB" )
})
```

***

## 📖 Related Pages

* [Audio Overview](README.md)
* [Speech-to-Text](speech-to-text.md)
* [Audio Translation](audio-translation.md)
* [aiSpeak BIF Reference](../../advanced/reference/built-in-functions/aispeak.md)
