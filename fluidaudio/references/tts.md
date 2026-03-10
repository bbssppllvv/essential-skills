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

Lightweight (~155M parameters) autoregressive TTS model with voice cloning support. Uses SentencePiece tokenization (no espeak dependency). Supports streaming generation and voice cloning from short audio samples (1-30 seconds).

### Basic Synthesis

```swift
import FluidAudio

let pocketTTS = PocketTTS()
let audioData = try await pocketTTS.synthesize(text: "Hello from FluidAudio")
try audioData.write(to: URL(fileURLWithPath: "output.wav"))
```

### Voice Cloning

Clone a voice from a 1-30 second audio sample:

```swift
import FluidAudio

let pocketTTS = PocketTTS()
let referenceAudio = URL(fileURLWithPath: "speaker_sample.wav")
let audioData = try await pocketTTS.synthesize(
    text: "This will sound like the reference speaker.",
    referenceAudio: referenceAudio
)
try audioData.write(to: URL(fileURLWithPath: "cloned_output.wav"))
```

### Streaming Output

```swift
import FluidAudio

let pocketTTS = PocketTTS()
for try await chunk in pocketTTS.synthesizeStreaming(text: "Stream this sentence as it is generated.") {
    // Each chunk is a Data fragment of audio, play or write incrementally
    audioPlayer.enqueue(chunk)
}
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
