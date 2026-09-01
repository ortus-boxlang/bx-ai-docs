---
description: Transcribe audio from a file path, URL, or binary data into text using an AI provider.
icon: closed-captioning
---

# aiTranscribe

Transcribe audio from a file path, URL, or binary data into text using an AI provider.

## Syntax

```javascript
aiTranscribe( audio, params={}, options={} )
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `audio` | string or binary | ✅ Yes | Audio source: local file path, public URL, or raw binary data |
| `params` | struct | No | Provider API parameters sent directly to the AI provider (e.g. `model`, `temperature`) |
| `options` | struct | No | Module-level behavior options (provider, return format, timestamps, etc.) |

### Audio Input Detection

The `audio` parameter is auto-detected:

| Input | How it is detected |
|---|---|
| File path | String ending with an audio extension (`.mp3`, `.wav`, `.m4a`, `.webm`, `.ogg`, `.flac`, etc.) |
| URL | String starting with `http://` or `https://` |
| Binary data | BoxLang binary / Java `byte[]` value |

## Params

| Param | Type | Default | Description |
|---|---|---|---|
| `model` | string | (config default) | Transcription model to use (e.g. `whisper-1`, `whisper-large-v3`) |
| `language` | string | `""` | BCP-47 language code of the audio (e.g. `en`, `es`, `fr`). Improves accuracy when known |
| `inputFormat` | string | (auto) | Audio format of binary input: `mp3`, `wav`, `flac`, `webm`, `ogg`, `m4a`, etc. Only required when passing raw `byte[]` data — file paths are auto-detected from their extension. Auto-seeded from `audio.defaultOutputFormat` in your module config when not specified |

## Options

| Option | Type | Default | Description |
|---|---|---|---|
| `provider` | string | (config default) | AI provider: `openai`, `groq` |
| `apiKey` | string | (env var) | Provider API key. Falls back to `<PROVIDER>_API_KEY` environment variable |
| `returnFormat` | string | `"text"` | `"text"` — returns plain string; `"response"` — returns `AiTranscriptionResponse` |
| `responseFormat` | string | `"json"` | Provider-level format: `json`, `text`, `verbose_json`, `srt`, `vtt` |
| `timestamps` | array | `[]` | Timestamp granularities: `["segment"]`, `["word"]`, or both |
| `diarize` | boolean | `false` | Enable speaker diarization (Groq only) |
| `timeout` | numeric | `90` | HTTP request timeout in seconds |
| `logRequest` | boolean | `false` | Log request to the module log file |
| `logRequestToConsole` | boolean | `false` | Print request payload to the console |
| `logResponse` | boolean | `false` | Log response to the module log file |
| `logResponseToConsole` | boolean | `false` | Print raw provider response to the console |

## Returns

| Condition | Returns |
|---|---|
| `returnFormat: "text"` (default) | **string** — transcribed text |
| `returnFormat: "response"` | **`AiTranscriptionResponse`** — full response object |

### `AiTranscriptionResponse` Methods

| Method | Returns | Description |
|---|---|---|
| `getText()` | string | The transcribed text |
| `hasText()` | boolean | `true` if text is non-empty |
| `getWordCount()` | numeric | Number of words in the transcription |
| `getFormattedDuration()` | string | Human-readable duration, e.g. `"1:23"` |
| `hasSegments()` | boolean | `true` if segment data is present |
| `hasWords()` | boolean | `true` if word-level timestamp data is present |
| `getSegments()` | array | Segment structs with `start`, `end`, `text` fields |
| `getWords()` | array | Word structs with `start`, `end`, `word` fields |
| `toStruct()` | struct | Full struct representation |
| `toJSON()` | string | JSON-serialized response |
| `toString()` | string | Alias for `getText()` |

## Output Formats

| `responseFormat` | Description |
|---|---|
| `json` | Default JSON with `text` field |
| `text` | Plain text only — fastest, no metadata |
| `verbose_json` | Full JSON with segments, words, language, duration |
| `srt` | SubRip subtitle format (.srt) |
| `vtt` | WebVTT subtitle format for HTML5 `<track>` elements |

> Pair `returnFormat: "response"` with `responseFormat: "verbose_json"` to populate timestamps, segments, and words.

## Events Fired

| Event | When |
|---|---|
| `beforeAITranscription` | Before the STT request is sent to the provider |
| `afterAITranscription` | After the STT response is received |

## Examples

### Basic — get plain text

```javascript
text = aiTranscribe( "/recordings/meeting.mp3" )
println( text )
```

### Full response with metadata

```javascript
result = aiTranscribe(
    "/recordings/meeting.mp3",
    {},
    { returnFormat: "response", responseFormat: "verbose_json" }
)

println( "Text: #result.getText()#" )
println( "Duration: #result.getFormattedDuration()#" )
println( "Words: #result.getWordCount()#" )
println( "Language: #result.language#" )
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
    println( "[#word.start#s – #word.end#s] #word.word#" )
})
```

### Specify language for better accuracy

```javascript
text = aiTranscribe(
    "/recordings/spanish-lecture.mp3",
    {},
    { language: "es" }
)
```

### Fast transcription with Groq

```javascript
text = aiTranscribe(
    "/recordings/podcast.mp3",
    { model: "whisper-large-v3" },
    { provider: "groq" }
)
println( text )
```

### Transcribe binary data with explicit format

When passing raw binary audio (e.g. loaded from disk with `fileReadBinary()` or piped from `aiSpeak()`), use `inputFormat` so the module writes the temp file with the correct extension:

```javascript
// Load a WAV file as bytes, then transcribe
wavBytes = fileReadBinary( "/recordings/interview.wav" )

text = aiTranscribe(
    audio  : wavBytes,
    params : { inputFormat: "wav" }
)
println( text )
```

```javascript
// Round-trip: TTS → STT — format is auto-seeded from audio.defaultOutputFormat
// but can be specified explicitly for clarity
speech = aiSpeak( "Hello from BoxLang!" )

text = aiTranscribe(
    audio  : speech.getAudioData(),
    params : { inputFormat: "mp3" }   // matches audio.defaultOutputFormat in config
)
println( text )
```

> 💡 **Tip:** If `audio.defaultOutputFormat` is set in your `config/boxlang.json` (e.g. `"mp3"`), `inputFormat` is seeded automatically. You only need to pass it explicitly when your binary data is in a different format.

### Transcribe from a URL

```javascript
text = aiTranscribe( "https://cdn.example.com/audio/announcement.mp3" )
println( text )
```

### Transcribe binary audio data

```javascript
binaryAudio = fileReadBinary( "/tmp/upload.webm" )
text = aiTranscribe( binaryAudio )
println( text )
```

### Generate SRT captions

```javascript
srt = aiTranscribe(
    "/video/presentation.mp4",
    {},
    { responseFormat: "srt" }
)
fileWrite( "/video/presentation.srt", srt )
```

## See Also

* [Speech-to-Text Guide](../../../main-components/audio/speech-to-text.md)
* [Audio Overview](../../../main-components/audio/README.md)
* [aiSpeak](aispeak.md)
* [aiTranslate](aitranslate.md)
