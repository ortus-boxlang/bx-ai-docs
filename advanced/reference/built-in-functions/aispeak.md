# aiSpeak

Convert text to natural-sounding speech audio using an AI provider.

## Syntax

```javascript
aiSpeak( text, params={}, options={} )
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `text` | string | ✅ Yes | The text content to synthesize into speech |
| `params` | struct | No | Provider API parameters sent directly to the AI provider (e.g. `model`, `voice`, `speed`) |
| `options` | struct | No | Module-level behavior options (provider, output format, file output, logging) |

## Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config default) | AI provider name: `openai`, `mistral`, `gemini`, `grok`, `elevenlabs` |
| `apiKey` | string | (env var) | Provider API key. Falls back to `<PROVIDER>_API_KEY` environment variable |
| `voice` | string | (config default) | Voice name or ID. Provider-specific; see voice reference table below |
| `outputFormat` | string | `mp3` | Audio format: `mp3`, `wav`, `flac`, `opus`, `pcm` |
| `speed` | numeric | `1.0` | Playback speed multiplier. Range: 0.25 – 4.0 |
| `outputFile` | string | `""` | When set, saves audio to this path and returns the file path string instead of `AiSpeechResponse` |
| `timeout` | numeric | `30` | HTTP request timeout in seconds |
| `logRequest` | boolean | `false` | Log request to the module log file |
| `logRequestToConsole` | boolean | `false` | Print request payload to the console |
| `logResponse` | boolean | `false` | Log response to the module log file |
| `logResponseToConsole` | boolean | `false` | Print raw provider response to the console (useful for debugging) |

## Returns

| Condition | Returns |
|---|---|
| `outputFile` **not** set | **`AiSpeechResponse`** — wraps binary audio data with convenience methods |
| `outputFile` **is** set | **string** — absolute path to the saved audio file |

### `AiSpeechResponse` Methods

| Method | Returns | Description |
|---|---|---|
| `saveToFile( filePath )` | string | Saves audio binary to the given path; returns absolute path |
| `getBase64()` | string | Base64-encoded audio data |
| `getMimeType()` | string | MIME type, e.g. `audio/mpeg` |
| `toDataURI()` | string | `data:audio/mpeg;base64,...` URI for HTML `<audio>` elements |
| `hasAudio()` | boolean | `true` if audio data is present and non-empty |
| `getSize()` | numeric | Audio data size in bytes |
| `getAudioFormat()` | string | Format string: `mp3`, `wav`, `flac`, etc. |
| `toStruct()` | struct | Metadata struct (no binary data — safe for logging) |
| `toJSON()` | string | JSON-serialized metadata |
| `getMetadataValue( key )` | any | Read a value from the response metadata bag |
| `setMetadataValue( key, value )` | this | Write a value to the metadata bag (chainable) |

## Events Fired

| Event | When |
|---|---|
| `beforeAISpeech` | Before the TTS request is sent to the provider |
| `afterAISpeech` | After the TTS response is received |

## Examples

### Synthesize and save to file

```javascript
audio = aiSpeak( "Welcome to BoxLang AI!" )
audio.saveToFile( expandPath( "/audio/welcome.mp3" ) )
println( "Size: #audio.getSize()# bytes" )
```

### Shorthand — write directly to disk with `outputFile`

```javascript
path = aiSpeak(
    "Your shipment has been dispatched.",
    {},
    { outputFile: expandPath( "/audio/notification.mp3" ) }
)
println( "Saved to: #path#" )
```

### Custom voice and format

```javascript
audio = aiSpeak(
    "Hello, this is the onyx voice at 1.5x speed.",
    { speed: 1.5 },
    { provider: "openai", voice: "onyx", outputFormat: "wav" }
)
audio.saveToFile( expandPath( "/audio/onyx.wav" ) )
```

### Base64 / data URI for web responses

```javascript
audio = aiSpeak( "Click here to play" )

// Embed in HTML
html = '<audio controls src="#audio.toDataURI()#"></audio>'

// Or return Base64 to a front-end client
response = { audio: audio.getBase64(), mimeType: audio.getMimeType() }
```

### ElevenLabs with voice ID

```javascript
audio = aiSpeak(
    "Bonjour, bienvenue dans BoxLang AI.",
    { voice_id: "21m00Tcm4TlvDq8ikWAM" },
    { provider: "elevenlabs" }
)
audio.saveToFile( expandPath( "/audio/french.mp3" ) )
```

### Custom interceptor for TTS logging

```javascript
BoxRegisterInterceptor( "afterAISpeech", function( event ) {
    var sizeKB = event.result.getSize() / 1024
    println( "TTS: #event.service.getName()# — #numberFormat( sizeKB, '0.0' )# KB" )
})
```

## Voice Reference

| Provider | Available Voices | Notes |
|---|---|---|
| **OpenAI** | `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer` | Default: `alloy` |
| **Mistral** | `Charlotte` | Only one voice |
| **Gemini** | `Kore` (and others) | See Gemini API docs for full list |
| **Grok / xAI** | `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer`, `eve` | OpenAI-compatible names |
| **ElevenLabs** | Voice IDs from your voice library | Pass as `voice_id` in params |

## See Also

* [Text-to-Speech Guide](../../../main-components/audio/text-to-speech.md)
* [Audio Overview](../../../main-components/audio/README.md)
* [aiTranscribe](aitranscribe.md)
* [aiTranslate](aitranslate.md)
