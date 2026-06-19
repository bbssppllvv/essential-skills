# Soniox Voice Agent & TTS Reference

## Table of Contents

- [TTS WebSocket API](#tts-websocket-api)
- [Voice Bot Demo](#voice-bot-demo)
- [VAD (Voice Activity Detection)](#vad-voice-activity-detection)
- [Tool Calling Pattern](#tool-calling-pattern)
- [Customization Guide](#customization-guide)
- [LangChain Integration](#langchain-integration)
- [Key Links](#key-links)

---

## TTS WebSocket API

**Endpoint:** `wss://tts-rt.soniox.com/tts-websocket`
**Model:** `tts-rt-v1-preview` (still tagged "preview" even after the GA launch on 2026-04-23 — only TTS model available)
**Auth:** `Authorization: Bearer …` header **or** `?api_key=…` query string. The `api_key` field is **not** part of the start config.

### Connection Flow

```python
import websockets, json, base64, uuid

url = f"wss://tts-rt.soniox.com/tts-websocket?api_key={API_KEY}"
async with websockets.connect(url) as ws:
    stream_id = f"tts-{uuid.uuid4()}"

    # 1. Send start config (no api_key field here)
    await ws.send(json.dumps({
        "model": "tts-rt-v1-preview",
        "language": "en",
        "voice": "Adrian",
        "audio_format": "pcm_s16le",
        "sample_rate": 24000,
        "stream_id": stream_id,
    }))

    # 2. Stream text chunks (can send while receiving audio)
    await ws.send(json.dumps({
        "text": "Hello, how can I help you today?",
        "text_end": False,
        "stream_id": stream_id,
    }))

    # 3. Signal end of text
    await ws.send(json.dumps({
        "text": "",
        "text_end": True,
        "stream_id": stream_id,
    }))

    # 4. Receive audio frames (per-frame `audio_end`, final message has `terminated: true`)
    async for message in ws:
        content = json.loads(message)
        if content.get("stream_id") != stream_id:
            continue
        if "audio" in content:
            audio_bytes = base64.b64decode(content["audio"])
            # Play or stream audio_bytes
        if content.get("terminated"):
            break
```

### TTS Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model` | string | — | `tts-rt-v1-preview` |
| `voice` | string | — | One of: Maya, Daniel, Noah, Nina, Emma, Jack, Adrian, Claire, Grace, Owen, Mina, Kenji |
| `language` | string | `en` | Language code (60+ supported, voices preserve identity across languages) |
| `audio_format` | string | `pcm_s16le` | `pcm_s16le`, `pcm_f32le`, `pcm_mulaw`, `pcm_alaw`, `wav`, `aac`, `mp3`, `opus`, `flac` |
| `sample_rate` | int | `24000` | 8000, 16000, 24000, 44100, 48000 |
| `bitrate` | int | — | 32000–320000, only for compressed formats |
| `stream_id` | string | auto | Unique ID per generation stream — auto-generated if omitted |

### Server Response Frames

| Field | Description |
|-------|-------------|
| `stream_id` | Echoes the client's `stream_id` |
| `audio` | Base64-encoded audio bytes for this frame |
| `audio_end` | `true` on the last audio frame for this stream's text run |
| `terminated` | `true` on the final message that closes the stream |
| `error_code`, `error_message` | Pre-stream errors only |

### REST TTS

Non-streaming alternative: `POST https://tts-rt.soniox.com/tts` with `Authorization: Bearer …` and JSON body `{model, language, voice, audio_format, text, sample_rate?, bitrate?}` — returns the raw audio bytes. For mid-stream errors REST surfaces `X-Tts-Error-Code` / `X-Tts-Error-Message` HTTP trailers.

### Pricing (current)

- $0.70/hr output audio (or $21.50 per 1M output audio tokens)
- $4.00 per 1M input text tokens

### Key TTS Behaviors

- **Streaming**: Audio generation starts before full text is received — send LLM chunks directly
- **Multiple streams**: Use unique `stream_id` per generation; ignore audio from cancelled streams
- **Cancellation**: No explicit cancel API — set `_active_stream_id = None` and ignore old stream audio

---

## Voice Bot Demo

**Repository:** `https://github.com/soniox/soniox_examples` → `apps/soniox-voice-bot-demo/`

### Architecture

```
Client (React/Twilio)
    ↕ WebSocket (binary audio + JSON messages)
Server (Python asyncio)
    ├── STT Processor → Soniox STT WebSocket (real-time transcription)
    ├── VAD Processor → Silero VAD (speech boundary detection)
    ├── LLM Processor → OpenAI-compatible API (streaming + tool calling)
    └── TTS Processor → Soniox TTS WebSocket (streaming speech synthesis)
```

### Message Flow

1. Client sends raw audio bytes via WebSocket
2. STT Processor streams audio to Soniox, emits `TranscriptionMessage` with tokens
3. STT detects `<end>` token → emits `TranscriptionEndpointMessage`
4. VAD detects speech start → emits `UserSpeechStartMessage` (cancels active LLM/TTS)
5. LLM Processor receives endpoint → generates streaming response with optional tool calls
6. TTS Processor receives LLM chunks → streams to Soniox TTS → sends audio back to client

### Server Files

- `main.py` — entry point. Default LLM is `os.getenv("OPENAI_MODEL", "gpt-5.4-mini")`.
- `session.py` — orchestrator: `asyncio.Queue` between processors; three concurrent tasks (`receive_messages`, `process_messages`, `send_messages`).
- `messages.py` — message types passed through queues.
- `tools.py` — system prompt + LLM tools (replace these for your use case).
- `processors/message_processor.py` — abstract base (`start()`, `process(message)`, `cleanup()`).
- `processors/stt.py` — talks to Soniox STT WebSocket. Note: the demo currently sets `"model": "stt-rt-preview"` rather than `stt-rt-v4` — switch to `stt-rt-v4` for new code.
- `processors/vad.py`, `processors/llm.py`, `processors/tts.py`.

### Twilio bridge

Separate sub-app at `apps/soniox-voice-bot-demo/twilio/` (its own `pyproject.toml`, `Dockerfile`, README). Bridges phone calls to the same backend over WebSocket. Use this when you want PSTN phone-number access instead of the React frontend.

### Setup

```bash
git clone https://github.com/soniox/soniox_examples.git
cd apps/soniox-voice-bot-demo/server

# Create .env (see .env.example for the full list)
cat > .env <<'EOF'
SONIOX_API_KEY=your_key
OPENAI_API_KEY=your_key
OPENAI_MODEL=gpt-5.4-mini
# Optional separate TTS endpoint/key:
# SONIOX_API_KEY_TTS=...
# SONIOX_API_HOST_TTS=tts-rt.soniox.com
EOF

# Install and run (requires Python 3.13+, uses uv)
uv run python main.py

# Frontend (separate terminal)
cd ../frontend
npm install && npm run dev
```

Dependencies (from `pyproject.toml`): `openai>=1.101.0`, `python-dotenv>=1.1.1`, `structlog>=25.4.0`, `websockets>=15.0.1`, `silero-vad`, `torch`, `torchaudio`, `onnxruntime`, `numpy`, `pydantic>=2.11.7`. The demo does **not** import the `soniox` Python SDK — it speaks raw WebSocket.

---

## VAD (Voice Activity Detection)

The voice bot uses **Silero VAD** (ONNX model) for detecting when the user starts/stops speaking.

```python
from silero_vad import VADIterator
import torch

torch.set_num_threads(1)
model, _ = torch.hub.load("snakers4/silero-vad", "silero_vad", onnx=True, trust_repo=True)

vad = VADIterator(model, sampling_rate=16000, threshold=0.5, min_silence_duration_ms=300)

# Process 512-sample chunks (16kHz) or 256-sample chunks (8kHz)
chunk_tensor = torch.from_numpy(audio_float32).unsqueeze(0)
result = vad(chunk_tensor, return_seconds=False)

if result and "start" in result:
    # User started speaking → cancel LLM/TTS
elif result and "end" in result:
    # User stopped speaking
```

### Why VAD matters for voice agents:
- **Barge-in**: User interrupts the bot mid-sentence → immediately cancel LLM generation and TTS playback
- **Faster than endpoint detection**: VAD triggers on raw audio before STT processes it
- **Separate from STT**: STT `<end>` token detects semantic endpoints; VAD detects acoustic silence

---

## Tool Calling Pattern

Tools are defined as OpenAI function tool params + async Python functions:

```python
from openai.types.chat import ChatCompletionFunctionToolParam

# 1. Define tool schema
my_tool_description = ChatCompletionFunctionToolParam(
    type="function",
    function={
        "name": "create_calendar_event",
        "description": "Creates a calendar event on the user's Mac",
        "parameters": {
            "type": "object",
            "properties": {
                "title": {"type": "string", "description": "Event title"},
                "date": {"type": "string", "description": "Date in YYYY-MM-DD"},
                "time": {"type": "string", "description": "Time in HH:MM"},
            },
            "required": ["title", "date", "time"],
        },
    },
)

# 2. Implement tool function
async def create_calendar_event(title: str, date: str, time: str) -> dict:
    result = subprocess.run(["osascript", "-e", f'''
        tell application "Calendar"
            tell calendar "Calendar"
                make new event with properties {{summary:"{title}", ...}}
            end tell
        end tell
    '''], capture_output=True, text=True)
    return {"success": True, "title": title, "date": date, "time": time}

# 3. Register in get_tools()
def get_tools():
    return [(my_tool_description, create_calendar_event)]
```

The LLM Processor handles the tool calling loop automatically:
1. LLM decides to call a tool → streams tool_call chunks
2. Processor collects tool call, executes the function
3. Appends tool result to messages
4. Calls LLM again with tool results → LLM generates final spoken response

---

## Customization Guide

### Changing the persona

Edit `get_system_message()` in `tools.py`:

```python
def get_system_message(language: str) -> str:
    return f"""
You are a helpful macOS assistant. You can control the user's computer,
create calendar events, manage files, check system performance, and more.
Keep responses short and conversational — this is a voice conversation.
Language: {language}
Current date: {datetime.now().isoformat()}
"""
```

### Using Claude instead of OpenAI

The LLM Processor uses the OpenAI SDK, which works with any OpenAI-compatible API:

```env
OPENAI_API_KEY=sk-ant-...
OPENAI_MODEL=claude-sonnet-4-7   # or current Sonnet/Haiku alias
OPENAI_BASE_URL=https://api.anthropic.com/v1/
```

Or use a proxy like LiteLLM that exposes Claude via OpenAI-compatible endpoint.

### Adding new tools

1. Define the tool schema (OpenAI function format)
2. Write the async function
3. Add the tuple to `get_tools()`
4. Update system prompt to mention the new tool

---

## LangChain Integration

**Package:** `langchain-soniox`

```bash
pip install langchain-soniox
```

```python
from langchain_soniox import SonioxDocumentLoader

# Transcribe audio file into a LangChain Document
loader = SonioxDocumentLoader(
    file_path="meeting.mp3",
    enable_speaker_diarization=True,
    language_hints=["en", "ru"],
)
docs = loader.load()
print(docs[0].page_content)  # Transcribed text
```

Note: This is async/batch transcription, not real-time. For real-time voice agents, use the WebSocket API directly (as in the voice bot demo).

---

## Key Links

| Resource | URL |
|----------|-----|
| Voice Bot Demo | `https://github.com/soniox/soniox_examples/tree/master/apps/soniox-voice-bot-demo` |
| Live Demo App | `https://github.com/soniox/soniox_examples/tree/master/apps/soniox-live-demo` |
| Python SDK | `https://github.com/soniox/soniox-python` |
| Node SDK | `https://github.com/soniox/soniox-js` |
| LangChain | `https://github.com/soniox/langchain-soniox` |
| Vercel AI SDK | `https://github.com/soniox/vercel-ai-sdk-provider` |
| n8n Nodes | `https://github.com/soniox/n8n-nodes-soniox` |
| Docs | `https://soniox.com/docs` |
| MCP Server | `npx -y mcp-remote https://soniox.com/docs/api/mcp/mcp` |
| LLM Context | `https://soniox.com/llms.txt` / `https://soniox.com/llms-full.txt` |
