# Soniox SDK Reference

## Table of Contents

- [Python SDK](#python-sdk) — `soniox`
- [Node SDK](#node-sdk) — `@soniox/node`
- [Web SDK](#web-sdk) — `@soniox/speech-to-text-web` (legacy) / `@soniox/client` (new)
- [React + React Native SDK](#react--react-native-sdk) — `@soniox/react`
- [Other Official SDKs](#other-official-sdks)
- [Error Classes](#error-classes)
- [Reconnection](#reconnection)

---

## Python SDK

### Installation

```bash
pip install soniox   # current: 2.3.0 (2026-04-23)
export SONIOX_API_KEY=<YOUR_API_KEY>
```

Region overrides (also available in Node): `SONIOX_REGION=eu|jp`, or full overrides `SONIOX_BASE_DOMAIN`, `SONIOX_API_BASE_URL`, `SONIOX_WS_URL`, `SONIOX_TTS_API_URL`, `SONIOX_TTS_WS_URL`.

### Client Initialization

**Sync** (scripts, CLIs):
```python
from soniox import SonioxClient
client = SonioxClient()
```

**Async** (FastAPI, aiohttp):
```python
from soniox import AsyncSonioxClient
client = AsyncSonioxClient()
```

Both expose identical namespaces (`stt`, `tts`, `files`, `models`, `auth`, `webhooks`, `realtime`). Async requires `await`.

### Real-Time Transcription

```python
from soniox import SonioxClient
from soniox.types import RealtimeSTTConfig, Token, StructuredContext, StructuredContextGeneralItem
from soniox.utils import render_tokens, start_audio_thread, throttle_audio

client = SonioxClient()

config = RealtimeSTTConfig(
    model="stt-rt-v4",
    audio_format="mp3",
    enable_endpoint_detection=True,
    enable_speaker_diarization=True,
    language_hints=["en"],
    context=StructuredContext(
        general=[StructuredContextGeneralItem(key="domain", value="meeting")],
        terms=["Soniox", "ASR"],
    ),
)

final_tokens: list[Token] = []
non_final_tokens: list[Token] = []

with client.realtime.stt.connect(config=config) as session:
    start_audio_thread(session, throttle_audio("audio.mp3", delay_seconds=0.1))

    for event in session.receive_events():
        for token in event.tokens:
            if token.is_final:
                final_tokens.append(token)
            else:
                non_final_tokens.append(token)
        print(render_tokens(final_tokens, non_final_tokens))
        non_final_tokens.clear()
```

**Streaming from URL:**
```python
from typing import Iterator
import httpx

def stream_audio_from_url(audio_url: str) -> Iterator[bytes]:
    with httpx.Client() as http:
        with http.stream("GET", audio_url) as response:
            response.raise_for_status()
            for chunk in response.iter_bytes(4096):
                if chunk:
                    yield chunk

with client.realtime.stt.connect(config=config) as session:
    start_audio_thread(session, stream_audio_from_url("https://stream.example.com/live.mp3"))
    for event in session.receive_events():
        pass  # process tokens
```

**Manual finalization** (push-to-talk):
```python
session.send_finalize()
```

**Endpoint detection** -- `<end>` token:
```python
for event in session.receive_events():
    for token in event.tokens:
        if token.text == "<end>":
            print("Speaker finished")
```

**Keepalive:**
```python
from soniox.utils import start_keep_alive_thread

session.send_keep_alive()  # single
thread, stop_event = start_keep_alive_thread(session, interval_seconds=10.0)  # background
```

**Temporary API key:**
```python
key = client.auth.create_temporary_api_key(
    expires_in_seconds=3600,
    client_reference_id="call-123",
)
print(key.api_key, key.expires_at)
```

### Async Transcription

In SDK v2.x the namespace is `client.stt` (the older `client.transcriptions` namespace was removed):

```python
from soniox import SonioxClient
client = SonioxClient()

# From local file
t = client.stt.transcribe(model="stt-async-v4", file="audio.mp3")

# From URL
t = client.stt.transcribe(
    model="stt-async-v4",
    audio_url="https://soniox.com/media/examples/coffee_shop.mp3",
)

# From uploaded file_id
t = client.stt.transcribe(model="stt-async-v4", file_id="uploaded-file-id")

# Poll, wait, retrieve
t = client.stt.get("transcription-id")
client.stt.wait("transcription-id")
transcript = client.stt.get_transcript("transcription-id")
print(transcript.text)
```

**List (paginated):**
```python
response = client.stt.list(limit=100)
for t in response.transcriptions:
    print(t.id, t.status)
while response.next_page_cursor:
    response = client.stt.list(limit=100, cursor=response.next_page_cursor)
```

**Delete:**
```python
client.stt.delete("transcription-id")
client.stt.destroy("transcription-id")  # + its uploaded file
client.stt.delete_all()
```

### TTS

REST one-shot:
```python
client.tts.generate_to_file(
    text="Hello there.",
    voice="Adrian",
    language="en",
    audio_format="mp3",
    output_path="hello.mp3",
)
```

Real-time streaming TTS (`tts-rt-v1-preview`):
```python
from soniox.types import RealtimeTTSConfig

with client.realtime.tts.connect(
    config=RealtimeTTSConfig(model="tts-rt-v1-preview", voice="Adrian", language="en",
                             audio_format="pcm_s16le", sample_rate=24000)
) as session:
    session.send_text("Hello, ", text_end=False)
    session.send_text("how are you?", text_end=True)
    for event in session.receive_events():
        play(event.audio)  # raw bytes
```

### Webhook Handling

**Configure on transcription:**
```python
from soniox.types import CreateTranscriptionConfig

config = CreateTranscriptionConfig(
    webhook_url="https://your-server.com/webhooks/soniox",
    webhook_auth_header_name="X-Webhook-Secret",
    webhook_auth_header_value="your-secret",
)
t = client.transcriptions.transcribe(audio_url="https://example.com/audio.mp3", config=config)
```

**Receive and verify (FastAPI):**
```python
from fastapi import FastAPI, Request
from soniox import SonioxClient
from soniox.errors import InvalidWebhookSignatureError
from soniox.types import WebhookAuthConfig

app = FastAPI()
client = SonioxClient()

@app.post("/webhooks/soniox")
async def soniox_webhook(request: Request):
    payload = await request.body()
    headers = dict(request.headers)
    try:
        event = client.webhooks.unwrap(
            payload, headers,
            auth=WebhookAuthConfig(name="X-Webhook-Secret", value="your-secret"),
        )
    except InvalidWebhookSignatureError:
        return {"error": "invalid signature"}
    if event.status == "completed":
        transcript = client.transcriptions.get_transcript(event.id)
        print(transcript.text)
```

### File Management

```python
# Upload (bytes, str/Path, or BinaryIO)
file = client.files.upload("audio.mp3")

# Get
file = client.files.get("file-id")
file = client.files.get_or_none("file-id")

# List (paginated)
response = client.files.list(limit=100)
while response.next_page_cursor:
    response = client.files.list(limit=100, cursor=response.next_page_cursor)

# Delete
client.files.delete("file-id")
client.files.delete_all()
```

---

## Node SDK

### Installation

```bash
npm install @soniox/node   # current: 2.0.0 (2026-04-23)
export SONIOX_API_KEY=<YOUR_API_KEY>
```

### Client Setup

```ts
import { SonioxNodeClient } from "@soniox/node";
const client = new SonioxNodeClient();
// Or with explicit options (note: snake_case keys):
const client2 = new SonioxNodeClient({ api_key: process.env.SONIOX_API_KEY });
```

### Real-Time Transcription

```ts
const session = client.realtime.stt({
    model: "stt-rt-v4",
});

session.on("result", (result) => {
    const text = result.tokens.map((t) => t.text).join("");
    if (text) console.log(text);
});

session.on("error", (err) => console.error("Error:", err));

await session.connect();

session.sendStream(stream, {
    pace_ms: 60,
    finish: true,
});
```

### Async Transcription

```ts
import { readFile } from "node:fs/promises";

const audio = await readFile("audio.mp3");
const transcription = await client.stt.transcribe({
    model: "stt-async-v4",
    file: audio,
    filename: "audio.mp3",
    wait: true,
});

console.log(transcription.transcript?.text);
```

---

## Web SDK

Soniox now ships **two** browser-capable SDKs:
- `@soniox/speech-to-text-web` — current STT-only browser SDK (1.4.0, 2026-01-13). Stable, recommended for STT-only browser apps.
- `@soniox/client` — newer unified browser/RN core (2.0.0, 2026-04-23). Same `SonioxClient` surface, snake_case option keys, also exposes `client.tts` for REST TTS. This is the base used by `@soniox/react`.

### Installation

```bash
npm install @soniox/speech-to-text-web   # legacy/STT-only
# OR
npm install @soniox/client                # newer, includes TTS
```

CDN:
```html
<script type="module">
  import { SonioxClient } from 'https://unpkg.com/@soniox/speech-to-text-web?module';
</script>
```

### SonioxClient

```ts
import { SonioxClient } from "@soniox/speech-to-text-web";

const sonioxClient = new SonioxClient({
    apiKey: "<SONIOX_API_KEY>",
    onError: (status, message) => console.error(status, message),
    onPartialResult: (result) => console.log(result.tokens),
});

sonioxClient.start({
    model: "stt-rt-v4",
    languageHints: ["en"],
    enableSpeakerDiarization: true,
    enableEndpointDetection: true,
    context: {
        general: [{ key: "domain", value: "Healthcare" }],
        terms: ["Celebrex", "Amoxicillin"],
    },
});

sonioxClient.stop();     // graceful, waits for final results
sonioxClient.cancel();   // immediate, discards session
sonioxClient.finalize(); // manual finalization
```

**Callbacks:** `onStarted`, `onFinished`, `onPartialResult(result)`, `onStateChange({newState, oldState})`, `onError(status, message)`

**Error statuses:** `get_user_media_failed`, `api_key_fetch_failed`, `queue_limit_exceeded`, `media_recorder_error`, `api_error`, `websocket_error`

### Temporary API Key Pattern

Avoid exposing API key in browser:

```ts
const sonioxClient = new SonioxClient({
    apiKey: async () => {
        const response = await fetch("/api/get-temporary-api-key", { method: "POST" });
        const { apiKey } = await response.json();
        return apiKey;
    },
});
```

### Custom Audio Streams

```ts
const audioElement = new Audio();
audioElement.crossOrigin = "anonymous";
audioElement.src = "https://example.com/audio.mp3";

const audioContext = new AudioContext();
const source = audioContext.createMediaElementSource(audioElement);
const destination = audioContext.createMediaStreamDestination();
source.connect(destination);
source.connect(audioContext.destination);

sonioxClient.start({
    model: "stt-rt-v4",
    stream: destination.stream,
});

audioElement.play();
```

---

## React + React Native SDK

Package: `@soniox/react` — current version 2.0.0 (2026-04-23). One package, both worlds: React, Next.js, **and React Native**. Built on top of `@soniox/client`.

```bash
npm install @soniox/react
```

There is **no separate `@soniox/react-native` package** — `@soniox/react` covers it. (The npm registry returns 404 for `@soniox/react-native` as of 2026-04-25.)

---

## Other Official SDKs

- **C# / .NET** — `github.com/soniox/soniox_csharp`. Older API surface; treat as legacy.
- No first-party SDK exists (as of 2026-04-25) for **Go, Rust, Java, or Swift**. Use raw WebSocket/REST.

## Error Classes

- Python: `SonioxApiError`, `SonioxRealtimeError`, `InvalidWebhookSignatureError`.
- Node / `@soniox/client`: `SonioxHttpError`, `SonioxRealtimeError`.

## Reconnection

The SDKs do **not** auto-reconnect on dropped WebSockets. If you need long-lived sessions, wrap `connect()` in your own retry loop and re-send the start config.
