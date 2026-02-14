# Infrastructure: Models, Audio, Platform

## Platform Requirements

- **macOS**: 14.0+ (Sonoma)
- **iOS**: 17.0+
- **Architecture**: arm64 only (Apple Silicon M/A-series). x86_64 (Intel) explicitly excluded.
- **Swift**: 5.9+
- **Xcode**: 16.0+ recommended
- **Frameworks**: CoreML, AVFoundation, Accelerate
- **Backend**: Apple Neural Engine (ANE) — less memory, faster inference

## Installation

### Swift Package Manager (recommended)

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/FluidInference/FluidAudio.git", from: "0.7.9")
]

targets: [
    .target(
        name: "YourTarget",
        dependencies: [
            .product(name: "FluidAudio", package: "FluidAudio")
        ]
    )
]
```

In Xcode: File → Add Package Dependencies → `https://github.com/FluidInference/FluidAudio.git`

### CocoaPods

```ruby
pod 'FluidAudio', '~> 0.7.8'
```

Note: CocoaPods uses `cocoapods-spm` for interop. Kokoro TTS is SPM-only.

### Cross-Platform Wrappers

- React Native/Expo: `@fluidinference/react-native-fluidaudio`
- Rust/Tauri: `fluidaudio-rs`

## Products & Licensing

| Product | Contents | License |
|---------|----------|---------|
| `FluidAudio` | ASR, Diarization, VAD, PocketTTS | Apache 2.0/MIT |
| `FluidAudioTTS` | + Kokoro TTS, ESpeakNG | GPL-3.0 |

Use `import FluidAudio` for core. Add `import FluidAudioTTS` only for Kokoro.

## Model Management

### Automatic Download Pipeline

1. Check local cache for `.mlmodelc` bundle
2. Download `.mlpackage` from HuggingFace if missing
3. Compile via `MLModel.compileModel()`
4. Validate `coremldata.bin` presence
5. Cache atomically for subsequent launches
6. Auto-recovery: re-download on corruption (3 retries, exponential backoff)

### Cache Locations

| Platform | Path |
|----------|------|
| macOS | `~/.cache/fluidaudio/Models/{repo-name}/` |
| iOS | `Library/Application Support/FluidAudio/Models/{repo-name}/` |

### Cache Structure Example

```
~/.cache/fluidaudio/Models/
├── parakeet-tdt-0.6b-v3-coreml/
│   ├── preprocessor.mlmodelc/
│   ├── encoder.mlmodelc/
│   ├── decoder.mlmodelc/
│   └── joint.mlmodelc/
├── silero-vad-coreml/
│   └── silero_vad.mlmodelc/
└── speaker-diarization-coreml/
    ├── segmentation_powerset.mlmodelc/
    └── wespeaker_embedding.mlmodelc/
```

### Model Repositories

| Component | Repository |
|-----------|-----------|
| ASR v3 | `FluidInference/parakeet-tdt-0.6b-v3-coreml` |
| ASR v2 | `FluidInference/parakeet-tdt-0.6b-v2-coreml` |
| Diarization | `FluidInference/speaker-diarization-coreml` |
| VAD | `FluidInference/silero-vad-coreml` |
| Kokoro TTS | `FluidInference/kokoro-82m-coreml` |
| PocketTTS | `FluidInference/pockettts-coreml` |

### Loading Methods

| Component | Method |
|-----------|--------|
| ASR | `AsrModels.downloadAndLoad(version:)` → `AsrModels` |
| Diarization (online) | `DiarizerModels.downloadIfNeeded()` → `DiarizerModels` |
| Diarization (offline) | `OfflineDiarizerManager.prepareModels()` → Void |
| VAD | `VadManager(config:)` → initialized manager |
| TTS (Kokoro) | `KokoroModel.synthesize(text:)` → Data |
| TTS (Pocket) | `PocketTTS.synthesize(text:)` → Data |

### Custom Registry & Proxy

```swift
// Redirect model downloads (before first manager init)
ModelRegistry.baseURL = "https://your-mirror.example.com"
```

```bash
# Environment variables
export REGISTRY_URL=https://your-mirror.example.com
export https_proxy=http://proxy.company.com:8080
export HF_TOKEN=hf_xxx
```

Xcode: Edit Scheme → Run → Arguments → Environment Variables → `REGISTRY_URL`

## AudioConverter

Central audio preprocessing. Converts any format to 16kHz mono Float32 once — all managers share the same buffer (zero-copy architecture).

```swift
let samples = try AudioConverter().resampleAudioFile(path: "audio.wav")
// Also: resampleAudioFile(at: URL)
```

Supported inputs: WAV, FLAC, ALAC (AVAudioFile), M4A, MP3 (AVAudioEngine), AVAudioPCMBuffer, [Float].

## ANE Optimization

After model loading:
1. `MLModel.load()` with `computeUnits: .all`
2. FP16 conversion (50% model size reduction)
3. 16-byte memory alignment for ANE
4. Lazy loading to reduce peak memory
5. LRU cache eviction

## Initialization Pattern (All Managers)

```swift
// 1. Download & load models
let models = try await AsrModels.downloadAndLoad(version: .v3)

// 2. Initialize manager
let manager = AsrManager(config: .default)
try await manager.initialize(models: models)

// 3. Process audio
let result = try await manager.transcribe(audioURL)
```

## Package Structure

```
FluidAudio/
├── Sources/
│   ├── FluidAudio/          # Core (ASR, Diarizer, VAD, Shared, Frameworks)
│   ├── FluidAudioTTS/       # Kokoro TTS
│   ├── FluidAudioCLI/       # CLI tool
│   ├── FastClusterWrapper/   # C++ hierarchical clustering
│   └── MachTaskSelfWrapper/  # C memory tracking
├── Tests/
└── Package.swift
```

## Concurrency

All managers support concurrent usage via Swift async/await. Model loading runs on background queues. Multiple manager instances can process in parallel.
