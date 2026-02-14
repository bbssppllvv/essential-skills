# Audio Inputs

Send audio files to speech-capable models on OpenRouter.

## Overview

OpenRouter enables sending audio files to compatible models through its API. Audio files must be **base64-encoded** - direct URLs are not supported for audio content.

Audio requests utilize the `/api/v1/chat/completions` endpoint with the `input_audio` content type. Only models featuring audio processing capabilities will handle these requests. You can identify suitable models by filtering for audio input modality on the [OpenRouter Models page](https://openrouter.ai/models).

## Supported Audio Formats

The platform supports multiple audio encoding types:

| Format   | Description      |
|----------|------------------|
| `wav`    | WAV audio        |
| `mp3`    | MP3 audio        |
| `aiff`   | AIFF audio       |
| `aac`    | AAC audio        |
| `ogg`    | OGG Vorbis audio |
| `flac`   | FLAC audio       |
| `m4a`    | M4A audio        |
| `pcm16`  | PCM16 audio      |
| `pcm24`  | PCM24 audio      |

> **Note**: Format support varies by provider and model. Check your model's documentation to confirm which audio formats it supports. Not all models support all formats.

## Code Examples

### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';
import fs from "fs/promises";

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

async function encodeAudioToBase64(audioPath: string): Promise<string> {
  const audioBuffer = await fs.readFile(audioPath);
  return audioBuffer.toString("base64");
}

// Read and encode the audio file
const audioPath = "path/to/your/audio.wav";
const base64Audio = await encodeAudioToBase64(audioPath);

const result = await openRouter.chat.send({
  model: "{{MODEL}}",
  messages: [
    {
      role: "user",
      content: [
        {
          type: "text",
          text: "Please transcribe this audio file.",
        },
        {
          type: "input_audio",
          inputAudio: {
            data: base64Audio,
            format: "wav",
          },
        },
      ],
    },
  ],
  stream: false,
});

console.log(result);
```

### Python

```python
import requests
import json
import base64

def encode_audio_to_base64(audio_path):
    with open(audio_path, "rb") as audio_file:
        return base64.b64encode(audio_file.read()).decode('utf-8')

url = "https://openrouter.ai/api/v1/chat/completions"
headers = {
    "Authorization": f"Bearer {API_KEY_REF}",
    "Content-Type": "application/json"
}

# Read and encode the audio file
audio_path = "path/to/your/audio.wav"
base64_audio = encode_audio_to_base64(audio_path)

messages = [
    {
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": "Please transcribe this audio file."
            },
            {
                "type": "input_audio",
                "input_audio": {
                    "data": base64_audio,
                    "format": "wav"
                }
            }
        ]
    }
]

payload = {
    "model": "{{MODEL}}",
    "messages": messages
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

### TypeScript (fetch)

```typescript
import fs from "fs/promises";

async function encodeAudioToBase64(audioPath: string): Promise<string> {
  const audioBuffer = await fs.readFile(audioPath);
  return audioBuffer.toString("base64");
}

// Read and encode the audio file
const audioPath = "path/to/your/audio.wav";
const base64Audio = await encodeAudioToBase64(audioPath);

const response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${API_KEY_REF}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "{{MODEL}}",
    messages: [
      {
        role: "user",
        content: [
          {
            type: "text",
            text: "Please transcribe this audio file.",
          },
          {
            type: "input_audio",
            input_audio: {
              data: base64Audio,
              format: "wav",
            },
          },
        ],
      },
    ],
  }),
});

const data = await response.json();
console.log(data);
```
