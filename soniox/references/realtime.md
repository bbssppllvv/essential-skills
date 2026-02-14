# Soniox Real-Time WebSocket Transcription Reference

Protocol-level reference for the Soniox real-time speech-to-text WebSocket API.

## Table of Contents

1. [WebSocket Endpoint & Connection](#1-websocket-endpoint--connection)
2. [Configuration Parameters](#2-configuration-parameters)
3. [Audio Formats](#3-audio-formats)
4. [Token Streaming Model](#4-token-streaming-model)
5. [Response Format](#5-response-format)
6. [Endpoint Detection](#6-endpoint-detection)
7. [Manual Finalization](#7-manual-finalization)
8. [Real-Time Translation](#8-real-time-translation)
9. [Connection Keepalive](#9-connection-keepalive)
10. [Ending the Stream](#10-ending-the-stream)
11. [Error Codes](#11-error-codes)
12. [Limits & Quotas](#12-limits--quotas)

---

## 1. WebSocket Endpoint & Connection

**Endpoint:** `wss://stt-rt.soniox.com/transcribe-websocket`

**Session lifecycle:**

1. Open WebSocket connection
2. Send JSON configuration (text frame) as first message
3. Stream audio as binary frames
4. Receive JSON token responses
5. Send empty frame to signal end-of-audio
6. Receive finished response; server closes connection

---

## 2. Configuration Parameters

First message must be a JSON text frame:

```json
{
  "api_key": "<SONIOX_API_KEY>",
  "model": "stt-rt-v4",
  "audio_format": "auto",
  "sample_rate": 16000,
  "num_channels": 1,
  "language_hints": ["en", "es"],
  "language_hints_strict": false,
  "enable_speaker_diarization": true,
  "enable_language_identification": true,
  "enable_endpoint_detection": true,
  "max_endpoint_delay_ms": 2000,
  "client_reference_id": "my-session-123",
  "context": {},
  "translation": {}
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `api_key` | string | yes | API key or temporary API key |
| `model` | string | yes | `stt-rt-v4` (recommended) |
| `audio_format` | string | yes | `"auto"` or raw encoding string |
| `sample_rate` | number | raw only | Sample rate in Hz (required for PCM) |
| `num_channels` | number | raw only | 1=mono, 2=stereo (required for PCM) |
| `language_hints` | string[] | no | ISO 639-1 codes to bias recognition |
| `language_hints_strict` | bool | no | Restrict to hinted languages only |
| `context` | object | no | Domain context (general, text, terms, translation_terms) |
| `enable_speaker_diarization` | bool | no | Adds `speaker` field to tokens |
| `enable_language_identification` | bool | no | Adds `language` field to tokens |
| `enable_endpoint_detection` | bool | no | Emits `<end>` token on speech completion |
| `max_endpoint_delay_ms` | number | no | 500-3000, default 2000 |
| `client_reference_id` | string | no | Tracking ID (max 256 chars) |
| `translation` | object | no | Translation config |

### Context Object

```json
{
  "context": {
    "general": [
      {"key": "domain", "value": "Healthcare"},
      {"key": "topic", "value": "Diabetes management"}
    ],
    "text": "Background text to prime the model...",
    "terms": ["Celebrex", "Zyrtec", "Amoxicillin"],
    "translation_terms": [
      {"source": "Mr. Smith", "target": "Sr. Smith"}
    ]
  }
}
```

Max context length: 10,000 characters.

---

## 3. Audio Formats

### Auto-detected (container formats)

Set `"audio_format": "auto"`. Supported: `aac`, `aiff`, `amr`, `asf`, `flac`, `mp3`, `ogg`, `wav`, `webm`.

### Raw PCM formats

Specify `audio_format`, `sample_rate`, and `num_channels`:

| Category | Formats |
|----------|---------|
| PCM signed | `pcm_s8`, `pcm_s16le`, `pcm_s16be`, `pcm_s24le`, `pcm_s24be`, `pcm_s32le`, `pcm_s32be` |
| PCM unsigned | `pcm_u8`, `pcm_u16le`, `pcm_u16be`, `pcm_u24le`, `pcm_u24be`, `pcm_u32le`, `pcm_u32be` |
| PCM float | `pcm_f32le`, `pcm_f32be`, `pcm_f64le`, `pcm_f64be` |
| Companded | `mulaw`, `alaw` |

---

## 4. Token Streaming Model

Responses contain **tokens** -- small text units (subwords, words, spaces, punctuation).

### Final vs Non-Final

| Property | `is_final: false` | `is_final: true` |
|----------|-------------------|------------------|
| Meaning | Provisional/speculative | Confirmed and stable |
| Mutability | May change or disappear | Never changes once emitted |
| Usage | Live feedback (captions) | Stable output, downstream processing |

**Pattern:** Append final tokens to accumulated list. Replace all non-final tokens with current response.

### Token Fields

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `text` | string | always | Token text |
| `is_final` | bool | always | Whether finalized |
| `start_ms` | number | spoken tokens | Start timestamp (ms) |
| `end_ms` | number | spoken tokens | End timestamp (ms) |
| `confidence` | number | always | 0.0-1.0 confidence score |
| `speaker` | string | if diarization | Speaker label ("1", "2", ...) |
| `language` | string | if lang ID | ISO language code |
| `translation_status` | string | if translation | "none", "original", or "translation" |
| `source_language` | string | translation only | Source speech language |

### Special Tokens

| Token | When | Meaning |
|-------|------|---------|
| `<end>` | Endpoint detected | Speaker finished utterance (always final) |
| `<fin>` | Manual finalization | Finalization complete (always final) |

---

## 5. Response Format

### Standard Response

```json
{
  "tokens": [
    {
      "text": "Hello",
      "start_ms": 600,
      "end_ms": 760,
      "confidence": 0.97,
      "is_final": true,
      "speaker": "1",
      "language": "en"
    }
  ],
  "final_audio_proc_ms": 760,
  "total_audio_proc_ms": 880
}
```

### Finished Response

```json
{
  "tokens": [],
  "final_audio_proc_ms": 1560,
  "total_audio_proc_ms": 1680,
  "finished": true
}
```

### Error Response

```json
{
  "tokens": [],
  "error_code": 503,
  "error_message": "Cannot continue request (code N). Please restart the request."
}
```

---

## 6. Endpoint Detection

Semantic detection of when a speaker finishes an utterance (intonation + pauses + context, not just VAD).

**Enable:** `"enable_endpoint_detection": true`

**Behavior:**
1. All preceding non-final tokens re-emitted as final
2. `<end>` token emitted (`is_final: true`)
3. Use `<end>` to trigger downstream logic

**Control delay:** `max_endpoint_delay_ms` (500-3000, default 2000). Lower = faster endpoint.

---

## 7. Manual Finalization

Force-finalize all pending non-final tokens:

```json
{"type": "finalize"}
```

**Behavior:**
1. All processed audio is finalized
2. All tokens returned with `is_final: true`
3. `<fin>` marker token emitted
4. Streaming may continue after finalization

**Guidelines:**
- Can call multiple times per session
- Wait ~200ms of silence after speech for best accuracy
- Don't call too frequently

---

## 8. Real-Time Translation

### One-Way Translation

Translate all speech into a single target language:

```json
{
  "translation": {
    "type": "one_way",
    "target_language": "fr"
  }
}
```

### Two-Way Translation

Bidirectional between two languages (A→B and B→A):

```json
{
  "translation": {
    "type": "two_way",
    "language_a": "en",
    "language_b": "de"
  }
}
```

### Translation Token Fields

| `translation_status` | Meaning |
|----------------------|---------|
| `"none"` | Not translated (outside translation pair) |
| `"original"` | Transcribed speech that will be translated |
| `"translation"` | Translated text (`source_language` indicates original) |

Spoken tokens include timestamps. Translation tokens do not.

---

## 9. Connection Keepalive

Send when not streaming audio to prevent timeout:

```json
{"type": "keepalive"}
```

- Send at least every **20 seconds** when idle
- 5-10 second interval recommended
- Session context preserved across keepalives

---

## 10. Ending the Stream

1. Send **empty WebSocket frame** (binary or text)
2. Server processes remaining audio, returns final responses
3. Server sends `finished` response
4. Server closes connection

---

## 11. Error Codes

| Code | Category | Common Messages |
|------|----------|----------------|
| **400** | Bad request | `Invalid model specified.`, `Missing audio format.`, `Audio decode error`, `Context is too long`, `Invalid language hint.` |
| **401** | Unauthorized | `Invalid API key.`, `Invalid/expired temporary API key.`, `Missing API key.` |
| **402** | Payment required | `Organization balance exhausted.`, `Organization monthly budget exhausted.` |
| **408** | Timeout | `Request timeout.`, `Input too slow`, `Start request timeout` |
| **429** | Rate limited | `Rate limit exceeded.`, `Max concurrent requests exceeded.` |
| **500** | Server error | Retry the request |
| **503** | Service unavailable | `Cannot continue request (code N).` -- reconnect immediately |

### Handling 503

Open a **new WebSocket connection** and restart streaming immediately.

---

## 12. Limits & Quotas

| Limit | Value |
|-------|-------|
| Requests per minute | 100 |
| Concurrent connections | 10 |
| Stream duration | 300 min/session |
| Context length | 10,000 chars |
| Client reference ID | 256 chars |

Higher limits (except stream duration) available via [Soniox Console](https://console.soniox.com/).

---

## Quick Reference: Client Messages

| Message | Format | Purpose |
|---------|--------|---------|
| Config | JSON text frame | First message, session setup |
| Audio | Binary frames | Streamed audio |
| Finalize | `{"type": "finalize"}` | Force-finalize tokens |
| Keepalive | `{"type": "keepalive"}` | Prevent timeout |
| End | Empty frame | Signal end-of-audio |

## Quick Reference: Server Messages

| Message | Key Fields | Description |
|---------|-----------|-------------|
| Tokens | `tokens`, `final_audio_proc_ms`, `total_audio_proc_ms` | Transcription results |
| Finished | `finished: true` | Session complete |
| Error | `error_code`, `error_message` | Error, connection closing |
