---
description: Translate audio from any language directly into English text using an AI provider.
icon: language
---

# aiTranslate

Translate audio from any language directly into English text using an AI provider. Unlike `aiTranscribe()`, the output language is always English regardless of the source audio language.

## Syntax

```javascript
aiTranslate( audio, params={}, options={} )
```

> **Note:** `aiTranslate()` performs **speech-to-English-text** translation in one step. It does not perform text-to-text translation between arbitrary languages. For audio transcription that preserves the original language, use [`aiTranscribe()`](aitranscribe.md).

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `audio` | string or binary | ✅ Yes | Audio source: local file path, public URL, or raw binary data |
| `params` | struct | No | Provider API parameters sent directly to the AI provider (e.g. `model`, `temperature`) |
| `options` | struct | No | Module-level behavior options (provider, return format, etc.) |

### Audio Input Detection

| Input | How it is detected |
|---|---|
| File path | String ending with an audio extension (`.mp3`, `.wav`, `.m4a`, `.webm`, `.ogg`, `.flac`, etc.) |
| URL | String starting with `http://` or `https://` |
| Binary data | BoxLang binary / Java `byte[]` value |

## Params

| Param | Type | Default | Description |
|---|---|---|---|
| `model` | string | (config default) | Translation model to use (e.g. `whisper-1`, `whisper-large-v3`) |
| `inputFormat` | string | (auto) | Audio format of binary input: `mp3`, `wav`, `flac`, `webm`, `ogg`, `m4a`, etc. Only required when passing raw `byte[]` data — file paths are auto-detected from their extension. Auto-seeded from `audio.defaultOutputFormat` in your module config when not specified |

## Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config default) | AI provider: `openai`, `groq` |
| `apiKey` | string | (env var) | Provider API key. Falls back to `<PROVIDER>_API_KEY` environment variable |
| `returnFormat` | string | `"text"` | `"text"` — returns plain English string; `"response"` — returns `AiTranscriptionResponse` |
| `responseFormat` | string | `"json"` | Provider-level format: `json`, `text`, `verbose_json`, `srt`, `vtt` |
| `timeout` | numeric | `30` | HTTP request timeout in seconds |
| `logRequest` | boolean | `false` | Log request to the module log file |
| `logRequestToConsole` | boolean | `false` | Print request payload to the console |
| `logResponse` | boolean | `false` | Log response to the module log file |
| `logResponseToConsole` | boolean | `false` | Print raw provider response to the console |

> The `language` option does not apply to `aiTranslate()`. The output is always English.

## Provider Support

| Provider | Supported |
|---|---|
| **OpenAI** | ✅ Yes |
| **Groq** | ✅ Yes |
| **Mistral** | ❌ No |
| **Gemini** | ❌ No |
| **ElevenLabs** | ❌ No |
| **Grok / xAI** | ❌ No |

## Returns

| Condition | Returns |
|---|---|
| `returnFormat: "text"` (default) | **string** — English text |
| `returnFormat: "response"` | **`AiTranscriptionResponse`** — full response object |

The returned `AiTranscriptionResponse` is the same object as returned by `aiTranscribe()`. See [aiTranscribe](aitranscribe.md) for the full method reference.

## Events Fired

| Event | When |
|---|---|
| `beforeAITranslation` | Before the translation request is sent to the provider |
| `afterAITranslation` | After the translation response is received |

## Examples

### Basic — translate any-language audio to English

```javascript
english = aiTranslate( "/recordings/french-presentation.mp3" )
println( english )
// "Good morning everyone. Today we will discuss quarterly results..."
```

### Transcribe vs translate side-by-side

```javascript
spanishAudio = "/recordings/hola-mundo.mp3"

// Transcribes — preserves original Spanish
spanish = aiTranscribe( spanishAudio )
println( "Original: #spanish#" )
// "Hola mundo, bienvenido a BoxLang AI."

// Translates — always returns English
english = aiTranslate( spanishAudio )
println( "English: #english#" )
// "Hello world, welcome to BoxLang AI."
```

### Full response object

```javascript
result = aiTranslate(
    "/recordings/german-lecture.mp3",
    {},
    { returnFormat: "response", responseFormat: "verbose_json" }
)

println( "English: #result.getText()#" )
println( "Duration: #result.getFormattedDuration()#" )
println( "Words: #result.getWordCount()#" )
```

### Fast translation with Groq

```javascript
english = aiTranslate(
    "/recordings/japanese-interview.mp3",
    { model: "whisper-large-v3" },
    { provider: "groq" }
)
println( english )
```

### Translate from a URL

```javascript
english = aiTranslate( "https://cdn.example.com/audio/portuguese-speech.mp3" )
println( english )
```

### Translate binary audio data

```javascript
binaryAudio = fileReadBinary( "/tmp/upload.m4a" )
english = aiTranslate( binaryAudio )
println( english )
```

## See Also

* [Audio Translation Guide](../../../main-components/audio/audio-translation.md)
* [Audio Overview](../../../main-components/audio/README.md)
* [aiTranscribe](aitranscribe.md)
* [aiSpeak](aispeak.md)
