# Text-to-Speech (TTS)

## Options

| Feature | PocketTTS | Kokoro |
|---------|-----------|--------|
| License | Apache 2.0 | GPL-3.0 |
| SPM Product | `FluidAudio` (core) | `FluidAudioTTS` |
| Quality | Standard | High |
| SSML | No | Yes |
| Phoneme control | No | Yes (via ESpeakNG) |
| Latency | ~80ms first-token | ~80ms per sentence |
| Architecture | arm64 only | arm64 only |

## PocketTTS (GPL-free)

```swift
import FluidAudio

let pocketTTS = PocketTTS()
let audioData = try await pocketTTS.synthesize(text: "Hello from FluidAudio")
try audioData.write(to: URL(fileURLWithPath: "output.wav"))
```

## Kokoro (High-quality, GPL-3.0)

```swift
import FluidAudio
import FluidAudioTTS

let kokoro = KokoroModel()
let audio = try await kokoro.synthesize(text: "Welcome to speech synthesis")

// SSML with phoneme control
let ssml = """
<speak>
    <phoneme alphabet="ipa" ph="təˈmɑːtəʊ">tomato</phoneme>
</speak>
"""
let audioWithPhonemes = try await kokoro.synthesize(text: ssml)
try audioWithPhonemes.write(to: URL(fileURLWithPath: "kokoro_output.wav"))
```

Kokoro supports: text normalization, phoneme generation via ESpeakNG, custom lexicon, SSML markup for prosody control.

Model: `FluidInference/kokoro-82m-coreml`

## CLI

```bash
swift run fluidaudio tts "Hello world" --output audio.wav
swift run fluidaudiocli tts --benchmark
```
