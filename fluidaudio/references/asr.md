# Automatic Speech Recognition (ASR)

## Models

| Version | Repository | Languages | Notes |
|---------|-----------|-----------|-------|
| v3 | `FluidInference/parakeet-tdt-0.6b-v3-coreml` | 25 European languages | Multilingual |
| v2 | `FluidInference/parakeet-tdt-0.6b-v2-coreml` | English only | Highest recall |

Architecture: Parakeet Token Duration Transducer (TDT) — 4 CoreML models: preprocessor, encoder, decoder, joint network.

## Performance

| Metric | Value | Hardware |
|--------|-------|----------|
| WER (v3) | 3.21% | LibriSpeech test-clean |
| RTFx | 155–209x | M4 Pro |
| Chunk processing | ~10ms | Per 14.96s segment |

## Batch Transcription

```swift
import FluidAudio

let models = try await AsrModels.downloadAndLoad(version: .v3)
let asrManager = AsrManager(config: .default)
try await asrManager.initialize(models: models)

// From URL
let result = try await asrManager.transcribe(audioURL)
print(result.text)

// From samples
let result2 = try await asrManager.transcribe(samples)  // [Float] 16kHz mono

// From buffer
let result3 = try await asrManager.transcribe(audioBuffer)  // AVAudioPCMBuffer
```

### ASRResult Fields

- `text`: Transcribed string
- `confidence`: Float 0.0–1.0
- `rtfx`: Real-time factor (processing time / audio duration)
- `tokenTimings`: Word-level timing information (optional)

### Configuration

```swift
let config = ASRConfig(
    chunkSizeSeconds: 14.96,    // Fixed chunk size
    chunkOverlapSeconds: 2.0     // Overlap for token boundary merging
)
let manager = AsrManager(config: config)
```

Chunk strategy: Fixed 14.96s chunks with 2s overlap for token boundary merging. Decoder state resets after each transcription for independent processing.

### Token Merging Strategies

- Contiguous: Adjacent token combining
- LCS (Longest Common Subsequence)
- Midpoint: Temporal boundary averaging

## Streaming ASR

### Core Classes

| Class | Purpose |
|-------|---------|
| `StreamingAsrManager` | Real-time processing without EOU |
| `StreamingEouAsrManager` | Streaming with end-of-utterance detection |

### Two-Tier Confirmation Model

- **Volatile updates**: Preliminary text that may change as more context arrives
- **Confirmed text**: Stable transcriptions once confidence thresholds are met

### StreamingEouAsrManager Configuration

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `chunkSeconds` | 11.0s | Main processing chunk size |
| `leftContextSeconds` | 2.0s | Historical context for continuity |
| `rightContextSeconds` | 2.0s | Lookahead for accuracy (adds latency) |
| `confirmationThreshold` | 0.80 | Confidence threshold for confirming text |
| `minContextForConfirmation` | 10.0s | Minimum audio before confirming any text |

EOU detection latency options: 160ms / 320ms / 1600ms windows.

### Streaming Usage

```swift
let streamingAsr = StreamingAsrManager(config: .streaming)
let models = try await AsrModels.downloadAndLoad(version: .v3)
try await streamingAsr.start(models: models)

// Consume updates asynchronously
Task {
    for await update in await streamingAsr.transcriptionUpdates {
        let type = update.isConfirmed ? "CONFIRMED" : "VOLATILE"
        print("[\(type)] \(update.text)")
    }
}

// Feed audio chunks
for audioChunk in audioBufferStream {
    await streamingAsr.streamAudio(audioChunk)
}

let finalText = try await streamingAsr.finish()
```

### End-of-Utterance Detection Strategies

1. **Silence-based**: Detects pauses exceeding configured thresholds
2. **Confidence-based**: Triggers when model confidence drops below threshold
3. **Context-based**: Confirms text when sufficient audio context accumulates

## Advanced Features

- **Vocabulary Boosting**: Custom vocabulary integration
- **CTC Rescoring**: Rescoring mechanism for accuracy improvement
- **Inverse Text Normalization** (text-processing-rs): Converts spoken-form to written form — "two hundred" → "200", "five dollars" → "$5". Available in Rust and Swift.

## CLI

```bash
swift run fluidaudio transcribe audio.wav
swift run fluidaudio transcribe audio.wav --model-version v2
swift run fluidaudiocli asr-benchmark --subset test-clean --max-files 100
swift run fluidaudiocli parakeet-eou --benchmark --chunk-size 160
swift run fluidaudiocli fleurs-benchmark --languages all --samples all
```
