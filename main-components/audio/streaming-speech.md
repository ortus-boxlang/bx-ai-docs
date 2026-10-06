---
description: >-
  Stream synthesized speech to a callback as it is generated, so you can
  restream it to a customer. Works with Cartesia, ElevenLabs, OpenAI, Mistral
  and Gemini.
icon: waveform-lines
---

# 🔊 Streaming Text-to-Speech

`aiSpeakStream()` sends text to a provider and calls your function with audio **as it arrives**, instead of waiting for the whole file. Use it to restream audio to a caller, a browser, or a telephony bridge.

```javascript
aiSpeakStream(
    "Thanks for calling BoxLang support.",
    ( event ) => {
        if ( event.type == "audio" ) {
            socket.send( event.data )   // binary audio, ready to forward
        }
    },
    { voice: "female" },
    { provider: "cartesia", outputFormat: "pcm" }
)
```

## 🔧 Syntax

```javascript
summary = aiSpeakStream( text, callback, params={}, options={} )

// Fluent
summary = aiSpeak().text( "Hello" ).provider( "cartesia" ).asPCM().stream( callback )
```

| Argument | Type | Description |
|---|---|---|
| `text` | string | The text to synthesize. Required and non-empty |
| `callback` | function | Called with one event struct per event (see below) |
| `params` | struct | Provider parameters: `model`, `voice`, `sample_rate`, `language`, and so on |
| `options` | struct | `provider`, `apiKey`, `outputFormat`, `timeout` (idle timeout in seconds), logging. `outputFile` is ignored when streaming |

## 📨 Callback events

Every event is a struct with a `type`:

| `type` | Fields | When |
|---|---|---|
| `audio` | `data` (binary), `format`, `sampleRate`, `sequence` | Each chunk of audio. `sampleRate` is `0` when the provider does not report one |
| `timestamps` | `words`, `start`, `end` | Cartesia over SSE when you pass `add_timestamps: true` |
| `done` | `chunks`, `bytes` | Always the last event, even when you stop early |

**Stop early.** Return an explicit `false` from the callback to stop (for example when the caller interrupts the agent). Any other return value, including nothing, keeps streaming. The `done` event is still sent and the summary reports `completed: false`.

```javascript
received = 0
summary = aiSpeakStream( "A long sentence...", ( event ) => {
    if ( event.type == "audio" ) {
        received++
        if ( received >= 3 ) return false
    }
}, {}, { provider: "cartesia", outputFormat: "mulaw" } )
println( summary.completed )  // false
```

Stopping closes the connection to the provider immediately, for both raw-byte and SSE streams, so generation stops and the final `done` event arrives right away.

## ⏲️ Timeout

`options.timeout` (default 30 seconds) is an **idle timeout**: the longest the stream will wait for the response headers or between received bytes, on both success and error responses. A provider that stalls is aborted with a `ProviderError`, while a healthy stream of any length is never cut off.

## 📋 Return value

A summary struct:

| Key | Description |
|---|---|
| `provider`, `model` | Who produced the audio |
| `audioFormat`, `sampleRate` | The format label placed on each chunk |
| `chunks`, `bytes` | Audio chunks and total bytes delivered |
| `completed` | `false` if your callback stopped the stream |

## 🧩 Provider support

| Provider | Streams | Transport | Formats | Notes |
|---|---|---|---|---|
| **Cartesia** | ✅ | `/tts/bytes` for mp3 and wav, `/tts/sse` for raw formats | `mp3`, `wav`, `pcm`, `mulaw`, `alaw` | SSE adds word timestamps. Force a transport with `params.transport` (`bytes` or `sse`). `mp3` and `wav` cannot use SSE |
| **ElevenLabs** | ✅ | `/stream` chunked bytes | `mp3`, `pcm`, `mulaw`, `alaw`, `opus` | `wav` cannot stream. Stream `pcm` instead |
| **OpenAI** | ✅ | Chunked bytes | Any `response_format`. `pcm` (24kHz) and `wav` start fastest | `params.stream_format` is ignored |
| **Mistral** | ✅ | SSE (`stream: true`) | `mp3`, `wav`, `pcm`, `flac`, `opus` | A UUID `voice` is sent as `voice_id` |
| **Gemini** | ✅ | Interactions API SSE | Raw 24kHz 16-bit mono PCM, labelled `pcm` | Headerless PCM. Wrap it yourself for a container |
| **Grok / xAI** | ❌ | WebSocket only (not implemented) | | Throws `UnsupportedCapability`. Use `aiSpeak()` |
| Others | ❌ | | | Throw `UnsupportedCapability` |

A provider can stream when `service.hasCapability( "speechStream" )` is true.

## 📞 Formats for voice agents

| Use case | Format | Why |
|---|---|---|
| Browser playback | `mp3` | Widely supported. Cartesia and ElevenLabs stream it |
| Voice agent pipeline | `pcm` | Raw, lowest latency, no decoding. Choose the rate with `params.sample_rate` (Cartesia default 24000) |
| Telephony (Twilio, SIP) | `mulaw` or `alaw` | 8kHz companded audio is what phone networks expect |

```javascript
// Telephony: 8kHz mulaw straight to the call
aiSpeakStream(
    "Your appointment is confirmed.",
    ( event ) => { if ( event.type == "audio" ) call.send( event.data ) },
    {},
    { provider: "cartesia", outputFormat: "mulaw" }
)
```

## ⏱️ Word timestamps (Cartesia)

```javascript
aiSpeakStream(
    "Hello streaming world.",
    ( event ) => {
        if ( event.type == "timestamps" ) {
            println( event.words.first() & " at " & event.start.first() & "s" )
        }
    },
    { add_timestamps: true },
    { provider: "cartesia", outputFormat: "pcm" }
)
```

## ⚠️ Errors

| Error type | When |
|---|---|
| `UnsupportedCapability` | The provider cannot stream speech |
| `InvalidArgument` | Empty text, an unsupported format for the provider, or an invalid transport |
| `ProviderError` | The provider rejected the request. The message includes the provider's explanation |

## 🔔 Events

`aiSpeakStream()` fires `beforeAISpeech` and `afterAISpeech` (with `stream: true` in the event data), plus `onAISpeakRequest` from the provider.

## See Also

- [aiSpeakStream reference](../../advanced/reference/built-in-functions/aispeakstream.md)
- [Text-to-Speech](text-to-speech.md)
