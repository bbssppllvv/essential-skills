# Video Inputs

Send video files to video-capable models on OpenRouter.

## Overview

OpenRouter enables sending video files to compatible models through the `/api/v1/chat/completions` API endpoint. The platform supports two video input methods:

- **Direct URLs** - Work efficiently for publicly accessible videos without requiring local encoding.
- **Base64 Data URLs** - Necessary for local files or private videos that aren't publicly accessible.

## Important Limitations

- Video URL support varies by provider. OpenRouter only sends video URLs to providers that explicitly support them.
- For instance, Google Gemini on AI Studio accepts only YouTube links, not Vertex AI.
- Video inputs are currently only supported via the API. Video uploads are not available in the OpenRouter chatroom interface at this time.

## Supported Video Formats

| Format | MIME Type    |
|--------|-------------|
| MP4    | video/mp4   |
| MPEG   | video/mpeg  |
| MOV    | video/mov   |
| WebM   | video/webm  |

## Best Practices

- **Video compression** reduces file size without significant quality loss.
- **Trim videos** to relevant segments.
- Consider **lower resolutions** (720p versus 4K) to reduce file size while maintaining usability.

For optimal results, balance video quality with practical considerations:

| Quality        | Resolution | Use Case                  |
|----------------|------------|---------------------------|
| High quality   | 1080p+     | Detailed analysis         |
| Medium quality | 720p       | General tasks             |
| Lower quality  | 480p       | Basic scene understanding |

## Provider-Specific Considerations

- **Google Gemini on AI Studio** supports only YouTube links.
- **Vertex AI** doesn't support video URLs and requires base64-encoded data URLs instead.

## Code Examples - Using Video URLs

### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

const result = await openRouter.chat.send({
  model: "{{MODEL}}",
  messages: [
    {
      role: "user",
      content: [
        {
          type: "text",
          text: "Please describe what's happening in this video.",
        },
        {
          type: "video_url",
          videoUrl: {
            url: "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
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

url = "https://openrouter.ai/api/v1/chat/completions"
headers = {
    "Authorization": f"Bearer {API_KEY_REF}",
    "Content-Type": "application/json"
}

messages = [
    {
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": "Please describe what's happening in this video."
            },
            {
                "type": "video_url",
                "video_url": {
                    "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
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
            text: "Please describe what's happening in this video.",
          },
          {
            type: "video_url",
            video_url: {
              url: "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
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

## Code Examples - Using Base64 Encoded Videos

### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';
import * as fs from 'fs';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

async function encodeVideoToBase64(videoPath: string): Promise<string> {
  const videoBuffer = await fs.promises.readFile(videoPath);
  const base64Video = videoBuffer.toString('base64');
  return `data:video/mp4;base64,${base64Video}`;
}

// Read and encode the video
const videoPath = 'path/to/your/video.mp4';
const base64Video = await encodeVideoToBase64(videoPath);

const result = await openRouter.chat.send({
  model: '{{MODEL}}',
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: "What's in this video?",
        },
        {
          type: 'video_url',
          videoUrl: {
            url: base64Video,
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
from pathlib import Path

def encode_video_to_base64(video_path):
    with open(video_path, "rb") as video_file:
        return base64.b64encode(video_file.read()).decode('utf-8')

url = "https://openrouter.ai/api/v1/chat/completions"
headers = {
    "Authorization": f"Bearer {API_KEY_REF}",
    "Content-Type": "application/json"
}

# Read and encode the video
video_path = "path/to/your/video.mp4"
base64_video = encode_video_to_base64(video_path)
data_url = f"data:video/mp4;base64,{base64_video}"

messages = [
    {
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": "What's in this video?"
            },
            {
                "type": "video_url",
                "video_url": {
                    "url": data_url
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
import * as fs from 'fs';

async function encodeVideoToBase64(videoPath: string): Promise<string> {
  const videoBuffer = await fs.promises.readFile(videoPath);
  const base64Video = videoBuffer.toString('base64');
  return `data:video/mp4;base64,${base64Video}`;
}

// Read and encode the video
const videoPath = 'path/to/your/video.mp4';
const base64Video = await encodeVideoToBase64(videoPath);

const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${API_KEY_REF}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: '{{MODEL}}',
    messages: [
      {
        role: 'user',
        content: [
          {
            type: 'text',
            text: "What's in this video?",
          },
          {
            type: 'video_url',
            video_url: {
              url: base64Video,
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
