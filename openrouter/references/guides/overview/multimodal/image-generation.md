# OpenRouter Image Generation Documentation

## Overview

OpenRouter enables image generation through models featuring `"image"` in their output modalities. Users can create images from text prompts by specifying appropriate modalities in API requests.

## Finding Image Generation Models

**On the Models Page**: Navigate to the [Models page](/models) and filter by output modalities to discover image-capable models.

**In the Chatroom**: The [Chatroom](/chat) offers an **Image** button that automatically filters and selects suitable models. The system will prompt you to add an image-capable model if none is currently active.

## API Implementation

Send requests to the `/api/v1/chat/completions` endpoint with the `modalities` parameter:

- **Text + Image Output** (e.g., Gemini): Use `modalities: ["image", "text"]`
- **Image-Only Output** (e.g., Flux, Sourceful): Use `modalities: ["image"]`

### Basic Image Generation Example

**Python**:
```python
import requests

url = "https://openrouter.ai/api/v1/chat/completions"
headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

payload = {
    "model": "google/gemini-2.5-flash-image-preview",
    "messages": [{"role": "user", "content": "Generate a sunset over mountains"}],
    "modalities": ["image", "text"]
}

response = requests.post(url, headers=headers, json=payload)
result = response.json()
```

**TypeScript**:
```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'google/gemini-2.5-flash-image-preview',
    messages: [
      {role: 'user', content: 'Generate a sunset over mountains'}
    ],
    modalities: ['image', 'text'],
  }),
});
```

## Image Configuration

### Aspect Ratio Options

Set `image_config.aspect_ratio` to specify dimensions:

| Ratio | Dimensions |
|-------|-----------|
| 1:1 | 1024x1024 (default) |
| 2:3 | 832x1248 |
| 3:2 | 1248x832 |
| 3:4 | 864x1184 |
| 4:3 | 1184x864 |
| 4:5 | 896x1152 |
| 5:4 | 1152x896 |
| 9:16 | 768x1344 |
| 16:9 | 1344x768 |
| 21:9 | 1536x672 |

### Image Size Control

Use `image_config.image_size` to adjust resolution:

- `1K` -> Standard resolution (default)
- `2K` -> Higher resolution
- `4K` -> Highest resolution

### Font Inputs (Sourceful Only)

The `image_config.font_inputs` parameter renders custom text with specific fonts. This works exclusively with Sourceful models (`sourceful/riverflow-v2-fast` and `sourceful/riverflow-v2-pro`).

**Structure**:
```json
{
  "image_config": {
    "font_inputs": [
      {
        "font_url": "https://example.com/fonts/custom-font.ttf",
        "text": "Hello World"
      }
    ]
  }
}
```

**Constraints**: Maximum 2 font inputs per request; $0.03 additional cost per input.

### Super Resolution References (Sourceful Only)

The `image_config.super_resolution_references` parameter enhances low-quality elements using reference images. This feature supports only Sourceful models during image-to-image generation.

**Structure**:
```json
{
  "image_config": {
    "super_resolution_references": [
      "https://example.com/reference1.jpg",
      "https://example.com/reference2.jpg"
    ]
  }
}
```

**Constraints**: Maximum 4 reference URLs; $0.20 per reference; requires input images in messages.

## Streaming Support

Image generation supports streaming by setting `"stream": True` (Python) or `stream: true` (TypeScript). The system returns image data through delta updates in the response stream.

## Response Structure

Generated images appear in the assistant message's `images` field:

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "I've generated a beautiful sunset image for you.",
        "images": [
          {
            "type": "image_url",
            "image_url": {
              "url": "data:image/png;base64,iVBORw0KGgo..."
            }
          }
        ]
      }
    }
  ]
}
```

**Image Characteristics**:
- Returned as base64-encoded data URLs
- Typically PNG format (`data:image/png;base64,`)
- Some models generate multiple images per request
- Dimensions vary by model capabilities

## Compatible Models

Examples include:
- `google/gemini-2.5-flash-image-preview`
- `black-forest-labs/flux.2-pro`
- `black-forest-labs/flux.2-flex`
- `sourceful/riverflow-v2-standard-preview`

## Best Practices

- Craft detailed, specific prompts for superior image quality
- Select models purpose-built for image generation
- Validate the `images` field exists before processing responses
- Account for potentially different rate limits for image generation
- Plan storage strategy for base64-encoded image data

## Troubleshooting

**Missing images in response?**

Confirm the model includes `"image"` in `output_modalities`. Verify correct `modalities` parameter usage. Check that your prompt requests image generation.

**Model unavailable?**

Use the Models page and filter by output modalities to locate compatible models.
