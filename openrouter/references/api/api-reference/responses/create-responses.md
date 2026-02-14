# Create a Response - OpenRouter API Documentation

## Overview

The Create a Response endpoint allows you to generate streaming or non-streaming responses using the OpenResponses API format.

**Endpoint:** `POST https://openrouter.ai/api/v1/responses`

## Authentication

Requests require an API key passed as a bearer token in the Authorization header:

```
Authorization: Bearer <your_api_key>
```

## Request Body

The endpoint accepts a JSON request body with the following structure:

### Core Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `input` | string \| array | Conversation history or prompt content |
| `model` | string | Model identifier (required) |
| `instructions` | string | System instructions for the model |
| `stream` | boolean | Enable streaming responses (default: false) |

### Model Configuration

| Parameter | Type | Default | Notes |
|-----------|------|---------|-------|
| `temperature` | number | — | Controls response randomness |
| `top_p` | number | — | Nucleus sampling parameter |
| `max_output_tokens` | number | — | Maximum response length |
| `top_k` | number | — | Top-k sampling parameter |
| `presence_penalty` | number | — | Penalizes repeated tokens |
| `frequency_penalty` | number | — | Frequency-based penalty |

### Advanced Features

**Tools & Functions:**
```json
{
  "tools": [{
    "type": "function",
    "name": "function_name",
    "description": "What the function does",
    "parameters": {
      "type": "object",
      "properties": {}
    }
  }],
  "tool_choice": "auto"
}
```

**Reasoning Configuration:**
```json
{
  "reasoning": {
    "effort": "high",
    "summary": "auto",
    "max_tokens": 1000
  }
}
```

**Response Format:**
```json
{
  "text": {
    "type": "text",
    "format": "json_object"
  }
}
```

### Provider Configuration

Control model routing and performance:

```json
{
  "provider": {
    "allow_fallbacks": true,
    "order": ["anthropic", "openai"],
    "max_price": {
      "prompt": "0.50",
      "completion": "1.00"
    }
  }
}
```

### Plugins

Enable optional features:

```json
{
  "plugins": [
    { "id": "web_search_2025_08_26", "enabled": true },
    { "id": "file-parser", "enabled": true },
    { "id": "auto-router", "enabled": true }
  ]
}
```

## Response Format

### Successful Response (200)

```json
{
  "id": "response_id",
  "object": "response",
  "created_at": 1234567890,
  "model": "anthropic/claude-4.5-sonnet-20250929",
  "status": "completed",
  "completed_at": 1234567900,
  "output": [
    {
      "id": "msg_id",
      "role": "assistant",
      "type": "message",
      "status": "completed",
      "content": [
        {
          "type": "output_text",
          "text": "Response content"
        }
      ]
    }
  ],
  "usage": {
    "input_tokens": 100,
    "output_tokens": 50,
    "total_tokens": 150
  }
}
```

### Response Status Values

- `completed` - Fully processed
- `incomplete` - Stopped early
- `in_progress` - Still processing
- `failed` - Error occurred
- `cancelled` - User cancelled
- `queued` - Awaiting processing

## Error Responses

| Status | Description |
|--------|-------------|
| 400 | Invalid request parameters |
| 401 | Authentication required or invalid |
| 402 | Insufficient credits |
| 404 | Resource not found |
| 408 | Request timeout |
| 413 | Payload exceeds size limits |
| 422 | Semantic validation failure |
| 429 | Rate limit exceeded |
| 500 | Server error |
| 502 | Provider/upstream API failure |
| 503 | Service temporarily unavailable |

## Code Examples

### Python
```python
import requests

url = "https://openrouter.ai/api/v1/responses"
payload = {
    "input": [{"type": "message", "role": "user", "content": "Hello"}],
    "model": "anthropic/claude-4.5-sonnet-20250929",
    "temperature": 0.7
}
headers = {
    "Authorization": "Bearer <token>",
    "Content-Type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)
print(response.json())
```

### JavaScript
```javascript
const url = 'https://openrouter.ai/api/v1/responses';
const options = {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <token>',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    input: [{type: "message", role: "user", content: "Hello"}],
    model: "anthropic/claude-4.5-sonnet-20250929",
    temperature: 0.7
  })
};

const response = await fetch(url, options);
const data = await response.json();
console.log(data);
```

### Go, Ruby, Java, PHP, C#, and Swift examples are also available in the original documentation.

## Input Message Types

Messages can include:
- `input_text` - Plain text content
- `input_image` - Images with detail level (auto, high, low)
- `input_file` - File content
- `input_audio` - Audio in MP3 or WAV format
- `input_video` - Video from URL or base64

## Output Content Types

Responses may contain:
- `output_text` - Text with optional annotations
- `function_call` - Tool invocation requests
- `web_search_call` - Web search operations
- `file_search_call` - File search operations
- `image_generation_call` - Image generation requests
- `reasoning` - Extended thinking output
