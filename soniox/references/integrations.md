# Soniox Integration Patterns & Third-Party Integrations

## Table of Contents

- [Architecture Patterns](#architecture-patterns)
- [Vercel AI SDK](#vercel-ai-sdk)
- [TanStack AI SDK](#tanstack-ai-sdk)
- [LiveKit](#livekit)
- [Pipecat](#pipecat)
- [LangChain](#langchain)
- [Twilio Integration](#twilio-integration)
- [n8n Integration](#n8n-integration)
- [Community Integrations](#community-integrations)
- [Data Residency](#data-residency)
- [Security and Compliance](#security-and-compliance)
- [AI Engineering](#ai-engineering)

---

## Architecture Patterns

### Direct Stream

**Flow:** Client --> Soniox WebSocket (temporary API key from server)

Lowest-latency. Browser sends audio directly to Soniox using temporary API key.

**Server endpoint (temp key):**
```js
app.get("/temporary-api-key", async (req, res) => {
  const response = await fetch("https://api.soniox.com/v1/auth/temporary-api-key", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.SONIOX_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ usage_type: "transcribe_websocket", expires_in_seconds: 60 }),
  });
  res.json(await response.json());
});
```

**Client:**
```js
import { RecordTranscribe } from "https://unpkg.com/@soniox/speech-to-text-web?module";

const { api_key } = await (await fetch("/temporary-api-key")).json();
const rt = new RecordTranscribe({ apiKey: api_key });

rt.start({
  model: "stt-rt-v4",
  languageHints: ["en"],
  onPartialResult: (result) => {
    for (const token of result.tokens) {
      // token.text, token.is_final
    }
  },
});
```

### Proxy Stream

**Flow:** Client --> Proxy Server (WebSocket) --> Soniox WebSocket

Use when you need server-side inspection, transformation, or storage.

**Proxy (Node.js):**
```js
const WebSocket = require("ws");
const wss = new WebSocket.Server({ server });

wss.on("connection", (ws) => {
  const queue = [];
  let ready = false;
  const soniox = new WebSocket("wss://stt-rt.soniox.com/transcribe-websocket");

  soniox.on("open", () => {
    soniox.send(JSON.stringify({
      api_key: process.env.SONIOX_API_KEY,
      audio_format: "auto",
      model: "stt-rt-v4",
    }));
    ready = true;
    while (queue.length > 0) soniox.send(queue.shift());
  });

  soniox.on("message", (data) => ws.send(data.toString()));
  ws.on("message", (data) => { ready ? soniox.send(data) : queue.push(data); });
  ws.on("close", () => soniox.close());
});
```

---

## Vercel AI SDK

**Package:** `@soniox/vercel-ai-sdk-provider`

```bash
npm install @soniox/vercel-ai-sdk-provider
```

```ts
import { soniox } from "@soniox/vercel-ai-sdk-provider";
import { experimental_transcribe as transcribe } from "ai";

const { text } = await transcribe({
  model: soniox.transcription("stt-async-v4"),
  audio: new URL("https://soniox.com/media/examples/coffee_shop.mp3"),
  providerOptions: {
    soniox: {
      languageHints: ["en", "es"],
      enableSpeakerDiarization: true,
      context: { terms: ["Soniox", "Vercel"] },
    },
  },
});
```

**Custom provider (regional endpoints):**
```ts
import { createSoniox } from "@soniox/vercel-ai-sdk-provider";
const soniox = createSoniox({
  apiKey: process.env.SONIOX_API_KEY,
  apiBaseUrl: "https://api.eu.soniox.com",
});
```

---

## TanStack AI SDK

**Package:** `@soniox/tanstack-ai-adapter`

```bash
npm install @soniox/tanstack-ai-adapter
```

```ts
import { generateTranscription } from "@tanstack/ai";
import { sonioxTranscription } from "@soniox/tanstack-ai-adapter";

const result = await generateTranscription({
  adapter: sonioxTranscription("stt-async-v4"),
  audio: new URL("https://soniox.com/media/examples/coffee_shop.mp3"),
  modelOptions: {
    enableSpeakerDiarization: true,
    languageHints: ["en"],
  },
});
```

**Raw tokens (translation/multilingual):**
```ts
const rawTokens = (result as any).providerMetadata?.soniox?.tokens;
// token: { text, start_ms, end_ms, language, translation_status, speaker, confidence }
```

---

## LiveKit

**Package:** `livekit-plugins-soniox` (PyPI). Use when building LiveKit Agents that need Soniox STT.

```bash
pip install "livekit-agents[soniox]" livekit-plugins-soniox
```

```python
from livekit.agents import AgentSession
from livekit.plugins import soniox

session = AgentSession(
    stt=soniox.STT(model="stt-rt-v4", language_hints=["en"]),
    # llm=..., tts=...
)
```

Docs: https://soniox.com/docs/integrations/livekit

---

## Pipecat

**Package:** `pipecat-ai[soniox]`. Provides `SonioxSTTService` for the Pipecat voice-pipeline framework.

```bash
pip install "pipecat-ai[soniox]"
```

```python
from pipecat.services.soniox.stt import SonioxSTTService

stt = SonioxSTTService(api_key=os.environ["SONIOX_API_KEY"], model="stt-rt-v4")
```

Docs: https://soniox.com/docs/integrations/pipecat

---

## LangChain

Two official packages:
- **Python:** `langchain-soniox` — `SonioxDocumentLoader` for batch transcription into LangChain `Document`s.
- **JavaScript/TypeScript:** see `https://soniox.com/docs/integrations/langchain/langchain-js`.

```bash
pip install langchain-soniox
```

```python
from langchain_soniox import SonioxDocumentLoader

loader = SonioxDocumentLoader(file_path="meeting.mp3", enable_speaker_diarization=True)
docs = loader.load()
```

This is async/batch transcription. For real-time, use the WebSocket API or LiveKit/Pipecat.

---

## Twilio Integration

Stream live phone call audio to Soniox. Architecture: Twilio --> Media Stream WebSocket --> Your Server --> Soniox WebSocket.

A working reference is `apps/soniox-voice-bot-demo/twilio/` inside `https://github.com/soniox/soniox_examples` (Python + Dockerfile + README). The standalone `soniox-twilio-realtime-transcription` repo has historically also existed; check the current docs at `https://soniox.com/docs/integrations/` for the canonical link before pinning.

**TwiML:**
```xml
<Response>
    <Start>
        <Stream url="wss://YOUR_SERVER/twilio" track="both_tracks" />
    </Start>
    <Dial>USER_PHONE_NUMBER</Dial>
</Response>
```

**Env:** `SONIOX_API_KEY`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_PHONE_NUMBER`, `WEBSOCKET_URL`, `USER_PHONE_NUMBER`.

---

## n8n Integration

**Package:** `@soniox/n8n-nodes-soniox`

Install in n8n: Settings > Community Nodes > `@soniox/n8n-nodes-soniox`

| Operation | Description |
|-----------|-------------|
| **Create** | Start transcription (binary file, URL, or file ID) |
| **Get Results** | Retrieve status/transcript |
| **Delete** | Delete transcription + file |

Features: polling mode, webhook mode, auto-delete, language hints, diarization, context, translation.

---

## Community Integrations

Listed at `https://soniox.com/docs/integrations/community-integrations`:
- **Agent Voice Response (AVR)** — Asterisk PBX bridge.
- **Unofficial Go SDK** — community-maintained Go bindings.

These are not first-party — verify maintenance status before relying on them in production.

---

## Data Residency

| Region | REST API | WebSocket |
|--------|----------|-----------|
| **US** | `api.soniox.com` | `stt-rt.soniox.com` |
| **EU** | `api.eu.soniox.com` | `stt-rt.eu.soniox.com` |
| **Japan** | `api.jp.soniox.com` | `stt-rt.jp.soniox.com` |

Audio and transcripts stay in region. System data (billing) may process outside.

---

## Security and Compliance

- **SOC 2 Type 2** -- ongoing independent security audits
- **GDPR** -- fully compliant
- **HIPAA** -- certified for PHI
- Audio/transcripts **never used to train models**
- **No retention** by default
- All communication encrypted with **TLS 1.2+**
- Stored data isolated per account

Contact: support@soniox.com for SOC 2 report, GDPR DPA, HIPAA BAA.

---

## AI Engineering

### MCP Server

```json
"soniox-docs": {
  "command": "npx",
  "args": ["-y", "mcp-remote", "https://soniox.com/docs/api/mcp/mcp"]
}
```

### LLM Context Files

- `https://soniox.com/llms.txt` -- core context
- `https://soniox.com/llms-full.txt` -- extended context
