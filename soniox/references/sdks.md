# Soniox SDK Reference

## Table of Contents

- [Python SDK](#python-sdk)
- [Node SDK](#node-sdk)
- [Web SDK](#web-sdk)

---

## Python SDK

### Installation

```bash
pip install soniox
export SONIOX_API_KEY=<YOUR_API_KEY>
```

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

Both expose identical APIs (`files`, `models`, `auth`, `webhooks`, `transcriptions`, `realtime`). Async requires `await`.

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

```python
from soniox import SonioxClient
client = SonioxClient()

# From local file
t = client.transcriptions.transcribe(model="stt-async-v4", file="audio.mp3")

# From URL
t = client.transcriptions.transcribe(
    model="stt-async-v4",
    audio_url="https://soniox.com/media/examples/coffee_shop.mp3",
)

# From uploaded file_id
t = client.transcriptions.transcribe(model="stt-async-v4", file_id="uploaded-file-id")

# Poll, wait, retrieve
t = client.transcriptions.get("transcription-id")
client.transcriptions.wait("transcription-id")
transcript = client.transcriptions.get_transcript("transcription-id")
print(transcript.text)
```

**List (paginated):**
```python
response = client.transcriptions.list(limit=100)
for t in response.transcriptions:
    print(t.id, t.status)
while response.next_page_cursor:
    response = client.transcriptions.list(limit=100, cursor=response.next_page_cursor)
```

**Delete:**
```python
client.transcriptions.delete("transcription-id")
client.transcriptions.destroy("transcription-id")  # + its uploaded file
client.transcriptions.delete_all()
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
npm install @soniox/node
export SONIOX_API_KEY=<YOUR_API_KEY>
```

### Client Setup

```ts
import { SonioxNodeClient } from "@soniox/node";
const client = new SonioxNodeClient();
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

### Installation

```bash
npm install @soniox/speech-to-text-web
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
    model: "stt-rt-preview",
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
    model: "stt-rt-preview",
    stream: destination.stream,
});

audioElement.play();
```
