---
description: >-
  Events fired around text-to-speech, speech-to-text, and audio translation
  requests.
icon: volume-high
---

# Audio Events

### beforeAISpeech

Fired before a text-to-speech request is sent to the provider.

**When**: Before `aiSpeak()` dispatches the HTTP request

#### Event Arguments

| Argument         | Type    | Description                              |
| ---------------- | ------- | ----------------------------------------- |
| `speechRequest`  | `Struct`| The full TTS request object              |
| `service`        | `any`   | The AI service provider instance         |

#### Example

```javascript
BoxRegisterInterceptor( "beforeAISpeech", function( event ) {
    println( "TTS request: provider=#event.service.getName()# chars=#event.speechRequest.text.len()#" )
})
```

***

### afterAISpeech

Fired after a text-to-speech response has been received from the provider.

**When**: After `aiSpeak()` receives and wraps the audio binary in `AiSpeechResponse`

#### Event Arguments

| Argument         | Type                | Description                              |
| ---------------- | -------------------- | ----------------------------------------- |
| `speechRequest`  | `Struct`            | The original TTS request object          |
| `service`        | `any`               | The AI service provider instance         |
| `result`         | `AiSpeechResponse`  | The response object with audio data      |

#### Example

```javascript
BoxRegisterInterceptor( "afterAISpeech", function( event ) {
    var sizeKB = event.result.getSize() / 1024
    println( "TTS complete: #event.service.getName()# — #numberFormat( sizeKB, '0.0' )# KB" )
})
```

***

### beforeAITranscription

Fired before a speech-to-text request is sent to the provider.

**When**: Before `aiTranscribe()` dispatches the HTTP request

#### Event Arguments

| Argument               | Type    | Description                              |
| ----------------------- | ------- | ----------------------------------------- |
| `transcriptionRequest` | `Struct`| The full STT request object              |
| `service`              | `any`   | The AI service provider instance         |

#### Example

```javascript
BoxRegisterInterceptor( "beforeAITranscription", function( event ) {
    println( "STT request: provider=#event.service.getName()#" )
})
```

***

### afterAITranscription

Fired after a speech-to-text response has been received from the provider.

**When**: After `aiTranscribe()` receives the transcription response

#### Event Arguments

| Argument               | Type                       | Description                              |
| ----------------------- | ---------------------------- | ----------------------------------------- |
| `transcriptionRequest` | `Struct`                   | The original STT request object          |
| `service`              | `any`                      | The AI service provider instance         |
| `result`               | `AiTranscriptionResponse`  | The transcription response object        |

#### Example

```javascript
BoxRegisterInterceptor( "afterAITranscription", function( event ) {
    println( "Transcribed: #event.result.getWordCount()# words via #event.service.getName()#" )
})
```

***

### beforeAITranslation

Fired before an audio translation request is sent to the provider.

**When**: Before `aiTranslate()` dispatches the HTTP request

#### Event Arguments

| Argument               | Type    | Description                                      |
| ----------------------- | ------- | -------------------------------------------------- |
| `transcriptionRequest` | `Struct`| The full translation request object              |
| `service`              | `any`   | The AI service provider instance                 |

#### Example

```javascript
BoxRegisterInterceptor( "beforeAITranslation", function( event ) {
    println( "Audio translation request: #event.service.getName()#" )
})
```

***

### afterAITranslation

Fired after an audio translation response has been received from the provider.

**When**: After `aiTranslate()` receives the English text response

#### Event Arguments

| Argument               | Type                       | Description                              |
| ----------------------- | ---------------------------- | ----------------------------------------- |
| `transcriptionRequest` | `Struct`                   | The original translation request object  |
| `service`              | `any`                      | The AI service provider instance         |
| `result`               | `AiTranscriptionResponse`  | The response with English text           |

#### Example

```javascript
BoxRegisterInterceptor( "afterAITranslation", function( event ) {
    println( "Translated: #event.result.getWordCount()# English words" )
})
```
