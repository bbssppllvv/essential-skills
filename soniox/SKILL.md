---
name: soniox
description: Integrate Soniox speech-to-text and text-to-speech APIs for real-time voice applications, voice agents, and transcription. Use when working with Soniox WebSocket streaming, Soniox REST transcription, Soniox TTS, Soniox SDKs (Python, Node, Web), building voice bots/agents with Soniox, or implementing voice/speech features using Soniox. Triggers on mentions of Soniox API, soniox package, @soniox/node, @soniox/speech-to-text-web, real-time transcription via Soniox, speech-to-text integration, text-to-speech with Soniox, Soniox voice bot, Soniox voice agent, or audio transcription with Soniox.
---

# Soniox Voice AI Platform

Cloud speech-to-text and text-to-speech APIs with real-time WebSocket streaming, async file transcription, and a complete voice agent framework. Supports 60+ languages, speaker diarization, live translation, and custom vocabulary context.

## API Structure

| API | Transport | Model | Use Case |
|-----|-----------|-------|----------|
| STT Real-Time | WebSocket `wss://stt-rt.soniox.com/transcribe-websocket` | `stt-rt-v4` | Live audio streaming, token-by-token results |
| STT Async | REST `https://api.soniox.com/v1/` | `stt-async-v4` | Pre-recorded files, batch processing |
| TTS Real-Time | WebSocket `wss://tts-rt.soniox.com/tts-websocket` | `tts-rt-v1-preview` | Streaming speech synthesis |

Authentication: `Authorization: Bearer <API_KEY>` header (or `api_key` query param for WebSocket).

## Quick Start — Real-Time WebSocket

1. Connect to `wss://stt-rt.soniox.com/transcribe-websocket?api_key=YOUR_KEY`
2. Send JSON config: `{"model": "stt-rt-v4", "audio_format": "pcm_s16le", "sample_rate": 16000}`
3. Stream raw audio bytes
4. Receive JSON tokens: `{"tokens": [{"text": "hello", "is_final": true, "start_ms": 100, "end_ms": 500}]}`
5. Close connection when done

Key token fields: `text`, `is_final` (false=provisional, true=confirmed), `start_ms`, `end_ms`, `confidence`, `speaker` (if diarization enabled), `language` (if language ID enabled).

## Quick Start — Async REST

The transcription endpoint is JSON-only. Source audio comes from a public URL or an uploaded `file_id` — there is no `audio_file` form field on `/v1/transcriptions`.

```bash
# Option A: public URL
curl -X POST https://api.soniox.com/v1/transcriptions \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"stt-async-v4","audio_url":"https://example.com/audio.mp3"}'

# Option B: upload file first (multipart), then create transcription
curl -X POST https://api.soniox.com/v1/files \
  -H "Authorization: Bearer $API_KEY" \
  -F file=@recording.mp3
# → returns {"id": "<file_id>", ...}

curl -X POST https://api.soniox.com/v1/transcriptions \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"stt-async-v4","file_id":"<file_id>"}'

# Poll until status="completed"; initial status is "queued"
curl https://api.soniox.com/v1/transcriptions/{id} \
  -H "Authorization: Bearer $API_KEY"

# Fetch the transcript itself
curl https://api.soniox.com/v1/transcriptions/{id}/transcript \
  -H "Authorization: Bearer $API_KEY"
```

## Configuration Options (Both APIs)

Common parameters sent in start config (real-time) or request body (async):

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | `stt-rt-v4` or `stt-async-v4` |
| `language_hints` | string[] | ISO 639-1 codes to improve accuracy |
| `language_hints_strict` | bool | Restrict recognition to hinted languages |
| `enable_language_identification` | bool | Detect language per token |
| `enable_speaker_diarization` | bool | Label speakers (up to 15) |
| `translation` | object | Translation config: `{"type": "one_way", "target_language": "fr"}` or `{"type": "two_way", "language_a": "en", "language_b": "fr"}` |
| `context` | object | Domain context (see below) |
| `max_endpoint_delay_ms` | int | 500-3000ms, semantic endpoint detection (real-time only) |

### Context Object Format

```json
{
  "context": {
    "general": [
      {"key": "domain", "value": "Healthcare"},
      {"key": "topic", "value": "Medical Consultation"}
    ],
    "text": "Background: Patient discussing cardiac symptoms...",
    "terms": ["myocardial infarction", "stent", "angioplasty"],
    "translation_terms": [
      {"source": "stent", "target": "стент"}
    ]
  }
}
```

Max 8000 tokens.

## Quick Start — TTS WebSocket

TTS went GA on 2026-04-23. The model `tts-rt-v1-preview` is still flagged "preview" in the docs but is the only available TTS model.

1. Connect to `wss://tts-rt.soniox.com/tts-websocket?api_key=YOUR_KEY` (or supply `Authorization: Bearer …` header — auth is **not** an inline field in the start config)
2. Send JSON start config: `{"model": "tts-rt-v1-preview", "voice": "Adrian", "language": "en", "audio_format": "pcm_s16le", "sample_rate": 24000, "stream_id": "tts-uuid"}` (`stream_id` is auto-generated if omitted)
3. Send text chunks: `{"text": "Hello world", "text_end": false, "stream_id": "tts-uuid"}`
4. Send final chunk: `{"text": "", "text_end": true, "stream_id": "tts-uuid"}`
5. Receive base64 audio frames: `{"stream_id": "tts-uuid", "audio": "base64...", "audio_end": false}`. The very last message of the stream sets `"terminated": true`.

Voice catalog (12 voices, all work across all 60+ languages with the same speaker identity): **Maya, Daniel, Noah, Nina, Emma, Jack, Adrian, Claire, Grace, Owen, Mina, Kenji**.

Audio formats: `pcm_s16le`, `pcm_f32le`, `pcm_mulaw`, `pcm_alaw`, `wav`, `aac`, `mp3`, `opus`, `flac`. Sample rates: 8000, 16000, 24000, 44100, 48000. `bitrate` (32000–320000) for compressed formats.

REST TTS also exists: `POST https://tts-rt.soniox.com/tts` with Bearer auth and JSON body `{model, language, voice, audio_format, text, sample_rate?, bitrate?}` — returns raw audio bytes.

TTS streams — audio generation starts before the full text is sent, enabling low-latency conversational responses when paired with LLM streaming.

## Voice Agent Architecture

Soniox provides a complete voice bot reference implementation: `soniox-voice-bot-demo` in the [soniox_examples](https://github.com/soniox/soniox_examples) repo.

### Pipeline

```
Microphone → STT Processor (Soniox WebSocket) → VAD (Silero) → LLM (OpenAI-compatible, streaming + tool calling) → TTS Processor (Soniox WebSocket) → Speaker
```

### Key components (Python server):

| File | Role |
|------|------|
| `session.py` | Orchestrates message flow between processors via async queues |
| `processors/stt.py` | Streams audio to Soniox STT, emits TranscriptionMessage + endpoint events |
| `processors/vad.py` | Silero VAD model, detects speech start/end, emits UserSpeechStart/EndMessage |
| `processors/llm.py` | OpenAI-compatible streaming chat + tool calling, cancellable on user speech |
| `processors/tts.py` | Streams LLM text to Soniox TTS, returns audio chunks, cancels on user interrupt |
| `tools.py` | Custom tools (functions) the LLM can call — **replace these for your use case** |

### Key patterns:
- **Barge-in**: When VAD detects user speech, LLM generation and TTS are cancelled immediately
- **Tool calling**: LLM calls tools defined in `tools.py`, gets results, then continues generating
- **Endpoint detection**: STT emits `<end>` token when speaker finishes — triggers LLM response
- **Metrics**: Tracks `llm_first_token_ms`, `tts_first_chunk_ms` for latency monitoring

### Dependencies:
The demo talks raw WebSocket — it does **not** use the `soniox` Python SDK. From `pyproject.toml`:
```
openai, python-dotenv, websockets, silero-vad, torch, torchaudio, onnxruntime, numpy, pydantic, structlog
```
Requires Python ≥3.13 and `uv`.

### Env:
```
SONIOX_API_KEY=...
OPENAI_API_KEY=...   # or any OpenAI-compatible API (Claude via proxy, etc.)
OPENAI_MODEL=gpt-5.4-mini   # current default in main.py
# Optional overrides for separate TTS endpoint/key:
SONIOX_API_KEY_TTS=...
SONIOX_API_HOST_TTS=tts-rt.soniox.com
```

There is also a `apps/soniox-voice-bot-demo/twilio/` sub-app that bridges phone calls to the same backend over WebSocket.

## Pricing

| Service | Cost |
|---------|------|
| STT Async | ~$0.10/hour |
| STT Real-Time | ~$0.12/hour |
| TTS Real-Time | ~$0.70/hour |

Pay-as-you-go, token-based. No free tier.

## Reference Files

Read these based on the specific task:

| File | When to Read |
|------|-------------|
| [references/realtime.md](references/realtime.md) | WebSocket protocol details, token streaming, finalization, keepalive, error codes |
| [references/async-api.md](references/async-api.md) | REST endpoints, file upload, job polling, webhooks, file management |
| [references/features.md](references/features.md) | Languages list, diarization details, context format, models, timestamps |
| [references/sdks.md](references/sdks.md) | Python/Node/Web SDK usage, code patterns, client initialization |
| [references/integrations.md](references/integrations.md) | Direct/Proxy stream patterns, Vercel AI, TanStack, Twilio, n8n, data residency, security |
| [references/voice-agent.md](references/voice-agent.md) | Voice bot demo architecture, TTS API details, VAD setup, tool calling patterns, customization guide |

## Native Swift/macOS Integration

Soniox has no native Swift SDK. For macOS/iOS apps, connect via raw WebSocket:

```swift
// URLSessionWebSocketTask approach
let url = URL(string: "wss://stt-rt.soniox.com/transcribe-websocket?api_key=\(apiKey)")!
let task = URLSession.shared.webSocketTask(with: url)
task.resume()

// Send start config
let config = """
{"model":"stt-rt-v4","audio_format":"pcm_s16le","sample_rate":16000}
"""
task.send(.string(config)) { error in /* handle */ }

// Stream audio bytes from microphone
task.send(.data(audioBuffer)) { error in /* handle */ }

// Receive tokens
func receiveNext() {
    task.receive { result in
        switch result {
        case .success(.string(let json)):
            // Parse tokens from JSON
            break
        case .failure(let error):
            // Handle error
            break
        default: break
        }
        receiveNext() // Continue receiving
    }
}
```

Audio format: Send raw PCM signed 16-bit little-endian at 16kHz mono for best results. The API also auto-detects encoded formats (mp3, ogg, flac, wav, etc.).

## Rate Limits

| Limit | Real-Time | Async |
|-------|-----------|-------|
| Requests/min | 100 | 100 |
| Concurrent | 10 connections | 100 pending jobs |
| Max duration | 300 min/session | — |
| Storage | — | 10GB, 1000 files |
| Total transcriptions | — | 2000 |

## Data Residency

Regional endpoints available:

| Region | Real-Time Endpoint | Async Endpoint |
|--------|-------------------|----------------|
| US (default) | `stt-rt.soniox.com` | `api.soniox.com` |
| EU | `stt-rt.eu.soniox.com` | `api.eu.soniox.com` |
| Japan | `stt-rt.jp.soniox.com` | `api.jp.soniox.com` |
