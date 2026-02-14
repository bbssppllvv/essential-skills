# Speaker Diarization

## Pipelines

| Pipeline | Manager | Clustering | DER | Mode |
|----------|---------|-----------|-----|------|
| Offline | `OfflineDiarizerManager` | VBx | 13.89% | Batch (full audio) |
| Online | `DiarizerManager` | Agglomerative | 17.7% | Streaming/real-time |
| Sortformer | — | End-to-end neural | — | Up to 4 speakers, no separate VAD/segmentation/clustering needed |

Both offline/online pipelines use Pyannote Community-1 architecture: segmentation model (powerset classification) + WeSpeaker embeddings.

Models: `FluidInference/speaker-diarization-coreml` (auto-download from HuggingFace).

RTFx: ~150x on M4 Pro.

## Offline Diarization (Best Accuracy)

```swift
import FluidAudio

let config = OfflineDiarizerConfig()
let manager = OfflineDiarizerManager(config: config)
try await manager.prepareModels()

// From URL
let result = try await manager.process(audioURL)

// From samples
let samples = try AudioConverter().resampleAudioFile(path: "meeting.wav")
let result2 = try await manager.process(audio: samples)

for segment in result.segments {
    print("Speaker \(segment.speakerId): \(segment.startTimeSeconds)s → \(segment.endTimeSeconds)s")
}
```

### Configuration

- `threshold`: Clustering similarity threshold (default: 0.7)
- `embeddingWindowSeconds`: Analysis window size (default: 1.5s)

## Online Diarization (Real-time)

```swift
import FluidAudio

let models = try await DiarizerModels.downloadIfNeeded()
let diarizer = DiarizerManager()
diarizer.initialize(models: models)

let samples = try AudioConverter().resampleAudioFile(path: "meeting.wav")
let result = try diarizer.performCompleteDiarization(samples)

for segment in result.segments {
    print("Speaker \(segment.speakerId): \(segment.startTimeSeconds)s - \(segment.endTimeSeconds)s")
}
```

### Configuration

- `clusteringThreshold`: Speaker similarity threshold (default: 0.7)
- `minSegmentDuration`: Minimum speech segment length (default: 0.25s)

## Result Structure

`DiarizationResult` contains:
- `segments`: Array of `DiarizationSegment`
  - `speakerId`: Speaker identifier
  - `startTimeSeconds` / `endTimeSeconds`: Temporal boundaries
  - Confidence scores
  - Optional speaker metadata (online mode)

## Sortformer

NVIDIA's end-to-end neural diarization model. Supports up to 4 speakers. No separate VAD, segmentation, or clustering required. Licensed under NVIDIA Open Model License.

See `Documentation/Diarization/Sortformer.md` in repo for streaming configuration and architecture details.

## CLI

```bash
swift run fluidaudio process meeting.wav --mode offline --threshold 0.6 --output results.json
swift run fluidaudio process meeting.wav --mode streaming
swift run fluidaudiocli diarization-benchmark --mode offline --auto-download --single-file ES2004a
swift run fluidaudio sortformer-benchmark --nvidia-high-latency
```

Options: `--threshold`, `--chunk-seconds` (5.0s default), `--overlap-seconds` (0.0s default).
