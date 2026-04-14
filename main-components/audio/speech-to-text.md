---
description: >-
  Transcribe audio files, URLs, or binary data into text using AI providers.
  aiTranscribe() auto-detects the input type and optionally returns rich
  metadata including word-level timestamps, segments, and duration.
icon: waveform-lines
---

# 🎧 Speech-to-Text

`aiTranscribe()` converts audio — from a local file path, a public URL, or raw binary data — into text. With basic usage you get a plain text string. With `returnFormat: "response"` you get a full `AiTranscriptionResponse` object containing timestamps, language detection, segments, word-level alignment, and more.

## 🔧 The `aiTranscribe()` Function

### Syntax

```javascript
aiTranscribe( audio, params={}, options={} )
```

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `audio` | string or binary | ✅ Yes | File path, URL, or raw binary audio data |
| `params` | struct | No | Provider API parameters (model, language, temperature, etc.) |
| `options` | struct | No | Module-level options (provider, returnFormat, responseFormat, etc.) |

### Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config) | AI provider: `openai`, `groq` |
| `apiKey` | string | (env var) | Provider API key |
| `returnFormat` | string | `"text"` | `"text"` — returns a plain string; `"response"` — returns `AiTranscriptionResponse` |
| `language` | string | `""` | Input audio language in BCP-47 format (e.g. `en`, `es`, `fr`). Optional but improves accuracy |
| `responseFormat` | string | `"json"` | Provider format: `json`, `text`, `verbose_json`, `srt`, `vtt` |
| `timestamps` | array | `[]` | Timestamp granularities: `["segment"]`, `["word"]`, or `["segment", "word"]` |
| `diarize` | boolean | `false` | Enable speaker diarization (Groq only) |
| `timeout` | numeric | `30` | HTTP timeout in seconds |
| `logRequest` | boolean | `false` | Log requests to the module log file |
| `logRequestToConsole` | boolean | `false` | Print request payload to console |
| `logResponse` | boolean | `false` | Log responses to the module log file |
| `logResponseToConsole` | boolean | `false` | Print raw provider response to console |

### Audio Input Detection

`aiTranscribe()` automatically detects the audio input type:

| Input | Detection Method |
|---|---|
| File path string | String ending with an audio extension (`.mp3`, `.wav`, `.m4a`, `.webm`, `.ogg`, `.flac`, etc.) |
| URL string | String beginning with `http://` or `https://` |
| Binary data | BoxLang binary / Java `byte[]` value |

## 📦 Return Value — `AiTranscriptionResponse`

By default (`returnFormat: "text"`), `aiTranscribe()` returns a plain **string** containing the transcribed text.

With `returnFormat: "response"`, it returns an **`AiTranscriptionResponse`** object.

### `AiTranscriptionResponse` Properties

| Property | Type | Description |
|---|---|---|
| `text` | string | Transcribed text |
| `segments` | array | Array of segment structs with start/end timestamps and text |
| `words` | array | Array of word structs with start/end timestamps |
| `language` | string | Detected or specified language code |
| `duration` | numeric | Total audio duration in seconds |
| `model` | string | Model used for transcription |
| `provider` | string | Provider name |
| `metadata` | struct | Raw provider response metadata |
| `timestamp` | datetime | When the transcription was created |

### `AiTranscriptionResponse` Methods

| Method | Returns | Description |
|---|---|---|
| `getText()` | string | Returns the transcribed text |
| `hasText()` | boolean | Returns `true` if transcribed text is non-empty |
| `getWordCount()` | numeric | Count of words in the transcription |
| `getFormattedDuration()` | string | Human-readable duration, e.g. `"1:23"` |
| `hasSegments()` | boolean | Returns `true` if segment data is available |
| `hasWords()` | boolean | Returns `true` if word-level timestamp data is available |
| `getSegments()` | array | Returns the array of segment structs |
| `getWords()` | array | Returns the array of word structs |
| `toStruct()` | struct | Returns a full struct representation |
| `toJSON()` | string | Returns JSON-serialized response |
| `toString()` | string | Alias for `getText()` |

## 🎼 Output Formats

When using `responseFormat` in options you can request different provider-level output styles:

| Format | Description |
|---|---|
| `json` | Default JSON with `text` field (minimal) |
| `text` | Plain text only — fastest, no metadata |
| `verbose_json` | Full JSON with segments, words, timestamps, language, duration |
| `srt` | SubRip subtitle format for video captioning |
| `vtt` | WebVTT subtitle format for HTML5 `<track>` elements |

> **Tip:** When using `returnFormat: "response"`, always pair it with `responseFormat: "verbose_json"` so word/segment data is populated.

## 💡 Examples

### Basic — transcribe and get plain text

```javascript
text = aiTranscribe( "/recordings/meeting.mp3" )
println( text )
```

### Full response object

```javascript
result = aiTranscribe(
    "/recordings/meeting.mp3",
    {},
    { returnFormat: "response", responseFormat: "verbose_json" }
)

println( "Text: #result.getText()#" )
println( "Language: #result.language#" )
println( "Duration: #result.getFormattedDuration()#" )
println( "Words: #result.getWordCount()#" )
```

### Word-level timestamps

```javascript
result = aiTranscribe(
    "/recordings/interview.wav",
    {},
    {
        returnFormat: "response",
        responseFormat: "verbose_json",
        timestamps: [ "word" ]
    }
)

result.getWords().each( function( word ) {
    println( "[#word.start#s–#word.end#s] #word.word#" )
})
```

### Specify language for better accuracy

```javascript
// Provide a BCP-47 language hint when you know the source language
text = aiTranscribe(
    "/recordings/spanish-lecture.mp3",
    {},
    { language: "es" }
)
```

### Groq — fast transcription

```javascript
// Groq's Whisper endpoint is significantly faster than OpenAI's
text = aiTranscribe(
    "/recordings/podcast.mp3",
    { model: "whisper-large-v3" },
    { provider: "groq" }
)
println( text )
```

### Transcribe from a URL

```javascript
text = aiTranscribe( "https://example.com/audio/announcement.mp3" )
println( text )
```

### Transcribe binary audio data

Useful when audio arrives in memory from an upload, a stream, or another API:

```javascript
// Read binary from an HTTP multipart upload or file
binaryAudio = fileReadBinary( "/tmp/upload.webm" )
text = aiTranscribe( binaryAudio )
println( text )
```

### Generate SRT captions for a video

```javascript
srt = aiTranscribe(
    "/video/presentation.mp4",
    {},
    { responseFormat: "srt" }
)
fileWrite( "/video/presentation.srt", srt )
```

## 📡 Events

| Event | Data Available |
|---|---|
| `beforeAITranscription` | `transcriptionRequest`, `service` |
| `afterAITranscription` | `transcriptionRequest`, `service`, `result` |

```javascript
// Track all transcription requests for cost monitoring
BoxRegisterInterceptor( "afterAITranscription", function( event ) {
    println( "Transcribed #event.result.getWordCount()# words via #event.service.getName()#" )
})
```

***

## 📖 Related Pages

* [Audio Overview](README.md)
* [Text-to-Speech](text-to-speech.md)
* [Audio Translation](audio-translation.md)
* [aiTranscribe BIF Reference](../../advanced/reference/built-in-functions/aitranscribe.md)
