---
description: >-
  Give agents audio editing, mastering, analysis and local speech tools by
  installing the bx-audio module next to BoxLang AI.
icon: sliders
---

# Audio Editing Tools (bx-audio)

`aiSpeak()`, `aiTranscribe()` and `aiTranslate()` call AI providers. The **bx-audio** module adds the other half: tools that *edit and measure* audio files on your machine, with no provider and no per-call cost. When both modules are installed, bx-audio registers 26 tools in the [Tool Registry](../tool-registry.md) under the module name `bxaudio`.

```bash
install-bx-module bx-ai
install-bx-module bx-audio
```

There is nothing to configure. bx-audio detects BoxLang AI when it loads, and does nothing if BoxLang AI is not there.

## Registered Is Not Given

Registering a tool only makes it available by name. An agent can call a tool only when its `tools` array names it, the same as every other registry tool.

```javascript
agent = aiAgent(
    name        : "sound-editor",
    instructions: "You edit audio files. Always work on copies and report the result's path.",
    tools       : [ "audioInfo@bxaudio", "audioTrim@bxaudio", "audioNormalize@bxaudio" ]
)

agent.run( "Cut the first 5 seconds off /recordings/intro.wav and normalize it" )
```

To hand an agent every audio tool, ask the registry for the module's tools:

```javascript
agent = aiAgent( name: "studio", tools: aiToolRegistry().getByModule( "bxaudio" ) )
```

> 💡 Give an agent only what its job needs. A transcription helper has no use for `audioMux`.

## How the Tools Behave

Agents cannot pass audio buffers, so every tool works on **files**:

* It reads a file by path.
* It writes a **new** file and returns `{ path, seconds, channels, sampleRate }`. Existing files are never overwritten.
* With no `outputPath`, the tool picks a name in an output folder, such as `intro-trim-3fa9c01b.wav`.
* Inspection tools return what they found as a struct or array.

An agent chains tools by passing each result's `path` to the next tool.

### File Access

Paths from a model are untrusted, so bx-audio resolves every path (absolute, `..` removed, symlinks followed) and only allows paths under an allowed folder. By default these are the system temp folder and the process working directory. Configure them in `boxlang.json`:

```json
{
  "modules": {
    "bxaudio": {
      "settings": {
        "ai": {
          "enabled": true,
          "allowedPaths": [ "/data/recordings" ],
          "outputDir": "/data/recordings/edited"
        }
      }
    }
  }
}
```

| Setting | Default | Description |
|---|---|---|
| `ai.enabled` | `true` | Set to `false` to stop bx-audio registering its tools |
| `ai.allowedPaths` | `[]` | Folders the tools may read and write. Empty means the temp folder and the working directory |
| `ai.outputDir` | `""` | Where results go when no output path is given. Empty means `bx-audio-ai` in the temp folder |

## The Tools

### Inspect

| Tool Key | Description |
|---|---|
| `audioInfo@bxaudio` | Channels, sample rate, length, and optionally peak and rms |
| `audioLoudness@bxaudio` | Integrated LUFS, loudness range and true peak (EBU R128) |
| `audioBpm@bxaudio` | Tempo, confidence and the first beat times |
| `audioDetectSilence@bxaudio` | The silent stretches as `{ start, end, duration }` |
| `audioReadTags@bxaudio` | Title, artist, album and other tags (read only) |
| `audioWaveformImage@bxaudio` | A PNG of the waveform |
| `audioSpectrogramImage@bxaudio` | A PNG of the spectrogram |
| `videoInfo@bxaudio` | Length and streams of any media file (needs FFmpeg) |

### Edit

| Tool Key | Description |
|---|---|
| `audioTrim@bxaudio` | Keep a section, from `start` to `end` seconds |
| `audioTrimSilence@bxaudio` | Cut the silence off both ends |
| `audioConcat@bxaudio` | Join files end to end |
| `audioMix@bxaudio` | Layer files so they play together |
| `audioFade@bxaudio` | Fade in and out |
| `audioStretch@bxaudio` | Change speed, keep the pitch |
| `audioPitch@bxaudio` | Change pitch in semitones, keep the length |
| `audioReverse@bxaudio` | Play it backwards |
| `audioGain@bxaudio` | Louder or quieter, in decibels |
| `audioConvert@bxaudio` | Convert to wav, mp3 or flac, with optional new sample rate and channels |

### Master

| Tool Key | Description |
|---|---|
| `audioNormalize@bxaudio` | Scale to a target peak level |
| `audioMaster@bxaudio` | Reach a loudness target (`streaming`, `podcast`, `broadcast` or LUFS) under a true-peak ceiling |
| `audioEq@bxaudio` | Parametric equalizer bands |
| `audioReverb@bxaudio` | Add reverb |

### Create

| Tool Key | Description |
|---|---|
| `audioVoices@bxaudio` | List the local speech voices |
| `audioSpeak@bxaudio` | Text to speech with a local Piper voice, no cloud service |
| `audioSilence@bxaudio` | Create silence of a given length |
| `audioMux@bxaudio` | Put an audio file on a video (needs FFmpeg) |

## Local or Cloud Speech

`speak@bxai` calls a cloud provider through `aiSpeak()`. `audioSpeak@bxaudio` runs a Piper voice on your machine, offline once the voice is downloaded. Name the one you want in the agent's `tools`, or both and let the model choose from their descriptions.

## Example: Podcast Clean Up

```javascript
agent = aiAgent(
    name        : "podcast-editor",
    instructions: "You edit podcast recordings. Work on copies, report loudness before and after.",
    tools       : [
        "audioLoudness@bxaudio", "audioTrimSilence@bxaudio",
        "audioEq@bxaudio", "audioMaster@bxaudio"
    ]
)

agent.run( "Take episode-12.wav, remove the silence at both ends, cut the rumble below 80 Hz and master it for podcast delivery." )
```

## Related Pages

* [Tool Registry](../tool-registry.md): Registry keys and the `getByModule()` method
* [Shipping Tools From Your Own Module](../../extending-boxlang-ai/custom-tools.md#shipping-tools-from-your-own-module): How bx-audio registers its tools
* [Text-to-Speech](text-to-speech.md): Provider-based speech with `aiSpeak()`
* [bx-audio guide](https://github.com/ortus-boxlang/bx-audio/blob/main/docs/guides/ai-tools.md): The full tool reference and troubleshooting
