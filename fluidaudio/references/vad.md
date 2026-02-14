# Voice Activity Detection (VAD)

## Model

- **Repository**: `FluidInference/silero-vad-coreml`
- **Architecture**: Silero VAD compiled to CoreML
- **Performance**: F1 0.85 @ 0.85 threshold (MUSAN/VOiCES benchmarks)
- **RTFx**: ~1000x+ (real-time capable)

## Batch Processing

```swift
import FluidAudio

let vadManager = VadManager(config: .default)
try await vadManager.prepareModels()

let segments = try await vadManager.segment(samples)
for segment in segments {
    print("Voice: \(segment.startTimeSeconds)s → \(segment.endTimeSeconds)s")
}
```

## Speech Segmentation

```swift
var segConfig = VadSegmentationConfig.default
segConfig.minSpeechDuration = 0.25
segConfig.minSilenceDuration = 0.4

let segments = try await vadManager.segmentSpeech(samples, config: segConfig)
```

## Streaming Mode

Maintains state across sequential chunks:

```swift
let streamState = VadStreamState()
for chunk in audioChunks {
    let segments = try await vadManager.segmentSpeech(chunk, streamingState: streamState)
}
```

## Configuration

### VadConfig

- `defaultThreshold`: Detection confidence 0.0–1.0 (default: 0.5)
- `computeUnits`: `.cpuAndNeuralEngine` (default) or `.cpuOnly`

### VadSegmentationConfig

- `minSpeechDuration`: Minimum speech segment length
- `minSilenceDuration`: Minimum silence gap for merging segments

## CLI

```bash
swift run fluidaudio vad audio.wav --output segments.json
swift run fluidaudiocli vad-benchmark --dataset voices-subset
swift run fluidaudiocli vad-analyze audio.wav --streaming --threshold 0.85
```
