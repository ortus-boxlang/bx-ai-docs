---
description: >-
  aiSpeakStream() streams synthesized speech to a callback as it is generated.
icon: waveform-lines
---

# aiSpeakStream

Stream text-to-speech audio from an AI provider as it is generated. See [Streaming Text-to-Speech](../../../main-components/audio/streaming-speech.md) for the full guide.

## Syntax

```javascript
summary = aiSpeakStream( text, callback, params={}, options={} )
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `text` | string | ✅ Yes | The text to synthesize. Must not be empty |
| `callback` | function | ✅ Yes | Called with each event struct. Return `false` to stop early and close the provider connection |
| `params` | struct | No | Provider parameters: `model`, `voice` (or `voice_id`), `sample_rate`, `language`, `add_timestamps`, `transport` (Cartesia) |
| `options` | struct | No | `provider`, `apiKey`, `outputFormat`, `speed`, `timeout` (idle timeout in seconds, default 30), logging |

## Callback events

| `type` | Fields |
|---|---|
| `audio` | `data` (binary), `format`, `sampleRate`, `sequence` |
| `timestamps` | `words`, `start`, `end` |
| `done` | `chunks`, `bytes` (always last) |

## Returns

A struct: `{ provider, model, audioFormat, sampleRate, chunks, bytes, completed }`. `completed` is `false` when the callback stopped the stream.

## Throws

| Type | When |
|---|---|
| `UnsupportedCapability` | The provider does not support streaming speech |
| `InvalidArgument` | Empty text, unsupported format, or invalid transport |
| `ProviderError` | The provider returned an error |

## Examples

```javascript
// Restream pcm to a client
aiSpeakStream(
    "Hello!",
    ( event ) => { if ( event.type == "audio" ) socket.send( event.data ) },
    { voice: "female" },
    { provider: "cartesia", outputFormat: "pcm" }
)

// Fluent builder
summary = aiSpeak()
    .text( "Hello!" )
    .provider( "elevenlabs" )
    .asMP3()
    .stream( ( event ) => { /* ... */ } )
```

## Providers

Cartesia, ElevenLabs, OpenAI, Mistral and Gemini stream. Grok does not (WebSocket only). Check with `aiService( "cartesia" ).hasCapability( "speechStream" )`.

## See Also

- [aiSpeak](aispeak.md)
- [Streaming Text-to-Speech](../../../main-components/audio/streaming-speech.md)
