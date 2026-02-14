# Create a Message

Create a message using the Anthropic Messages API format

---

**Endpoint:** `POST https://openrouter.ai/api/v1/messages`

**Content-Type:** `application/json`

This endpoint creates messages using the Anthropic Messages API format, supporting text, images, PDFs, tools, and extended thinking capabilities.

## Authentication

Include your API key as a bearer token in the `Authorization` header:

```
Authorization: Bearer <OPENROUTER_API_KEY>
```

## Request Schema

### Required Fields

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model identifier (e.g., `anthropic/claude-sonnet-4`) |
| `max_tokens` | number | Maximum number of tokens in the response |
| `messages` | array | Array of message objects with `role` and `content` |

### Optional Fields

| Parameter | Type | Description |
|-----------|------|-------------|
| `system` | string \| array | System instructions (string or array of system message objects) |
| `temperature` | number | Sampling temperature (0-1) |
| `top_p` | number | Nucleus sampling parameter |
| `top_k` | number | Token selection limit |
| `stop_sequences` | array | Stop generation at these strings |
| `stream` | boolean | Enable streaming responses |
| `tools` | array | Tool definitions (custom, bash, text_editor, web_search) |
| `tool_choice` | object | Tool selection strategy (`auto`, `any`, `none`, or specific tool) |
| `thinking` | object | Extended thinking configuration |
| `service_tier` | string | `"auto"` or `"standard_only"` |
| `metadata` | object | Contains `user_id` field for tracking |
| `provider` | object | Routing preferences and constraints |
| `plugins` | array | Feature plugins (auto-router, moderation, web, file-parser, response-healing) |
| `user` | string | End-user identifier (max 128 chars) |
| `session_id` | string | Request grouping ID (max 128 chars) |
| `output_config` | object | Effort level specification |

## Message Structure

Each message in the `messages` array contains:

- `role`: Either `"user"` or `"assistant"`
- `content`: Can be a string or an array of content blocks

### Basic Request Example

```json
{
  "model": "anthropic/claude-sonnet-4",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": "Hello, how are you?"
    }
  ]
}
```

## Supported Content Types

Messages support multiple content block types:

### 1. Text

```json
{
  "type": "text",
  "text": "What is the meaning of life?",
  "citations": [],
  "cache_control": null
}
```

### 2. Image

Images can be provided via base64 or URL (supports JPEG, PNG, GIF, WebP):

```json
{
  "type": "image",
  "source": {
    "type": "base64",
    "media_type": "image/jpeg",
    "data": "<base64-encoded-data>"
  }
}
```

Or via URL:

```json
{
  "type": "image",
  "source": {
    "type": "url",
    "url": "https://example.com/image.jpg"
  }
}
```

### 3. Document

PDFs and text documents with citation support:

```json
{
  "type": "document",
  "source": {
    "type": "base64",
    "media_type": "application/pdf",
    "data": "<base64-encoded-data>"
  },
  "title": "My Document",
  "citations": {"enabled": true}
}
```

### 4. Tool Use

```json
{
  "type": "tool_use",
  "id": "tool_use_123",
  "name": "get_weather",
  "input": {"location": "San Francisco"}
}
```

### 5. Tool Result

```json
{
  "type": "tool_result",
  "tool_use_id": "tool_use_123",
  "content": "72F and sunny"
}
```

### 6. Thinking

Extended thinking blocks with signatures:

```json
{
  "type": "thinking",
  "thinking": "Let me analyze this step by step...",
  "signature": "<signature-string>"
}
```

### 7. Redacted Thinking

Encrypted thinking content:

```json
{
  "type": "redacted_thinking",
  "data": "<encrypted-data>"
}
```

### 8. Server Tool Use

Server-side tool invocations like web search:

```json
{
  "type": "server_tool_use",
  "id": "srvtoolu_123",
  "name": "web_search",
  "input": {"query": "latest news"}
}
```

### 9. Web Search Tool Result

```json
{
  "type": "web_search_tool_result",
  "tool_use_id": "srvtoolu_123",
  "content": [
    {
      "type": "search_result",
      "url": "https://example.com",
      "title": "Example Result",
      "snippet": "This is a search result snippet..."
    }
  ]
}
```

## Tool Definitions

### Custom Tools

```json
{
  "type": "custom",
  "name": "get_weather",
  "description": "Get the current weather in a given location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "The city and state, e.g. San Francisco, CA"
      }
    },
    "required": ["location"]
  }
}
```

### Built-in Tools

- **`bash_20250124`**: Execute shell commands
- **`text_editor_20250124`**: File manipulation (str_replace_editor)
- **`web_search_20250305`**: Web searching with location/domain filtering

## Tool Choice Options

Control how the model uses tools:

- **`auto`**: Model decides whether to use tools (with optional parallel disabling)
- **`any`**: Model must use a tool (with optional parallel disabling)
- **`none`**: Disable tool usage
- **`tool`**: Force a specific named tool (with optional parallel disabling)

## Extended Thinking Configuration

```json
{
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000
  }
}
```

Supported types:

- `"enabled"`: Enable extended thinking with a token budget
- `"disabled"`: Disable extended thinking
- `"adaptive"`: Let the model decide when to use thinking

## Cache Control

Content blocks support ephemeral caching:

```json
{
  "cache_control": {
    "type": "ephemeral",
    "ttl": "5m"
  }
}
```

Supported TTL values: `"5m"`, `"1h"`

## Provider Configuration

The `provider` object enables routing preferences:

```json
{
  "provider": {
    "allow_fallbacks": true,
    "require_parameters": false,
    "data_collection": "deny",
    "zdr": false,
    "order": ["anthropic"],
    "only": ["anthropic"],
    "ignore": [],
    "quantizations": ["int4", "fp8"],
    "max_price": {
      "prompt": 0.01
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `allow_fallbacks` | boolean | Allow fallback to other providers |
| `require_parameters` | boolean | Require all parameters to be supported |
| `data_collection` | string | `"deny"` or `"allow"` data collection |
| `zdr` | boolean | Zero data retention |
| `order` | array | Preferred provider order |
| `only` | array | Restrict to specific providers |
| `ignore` | array | Exclude specific providers |
| `quantizations` | array | Acceptable quantization levels |
| `max_price` | object | Maximum price constraints |

## Plugins

The `plugins` array enables optional features:

| Plugin | Description |
|--------|-------------|
| `auto-router` | Automatic model routing with optional filtering |
| `moderation` | Content moderation checks |
| `web` | Web search with configurable engine and result limits |
| `file-parser` | Advanced PDF parsing with engine selection |
| `response-healing` | Response correction and optimization |

## Response Schema

### Successful Response (200)

```json
{
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! I'm doing well, thank you for asking.",
      "citations": []
    }
  ],
  "model": "anthropic/claude-sonnet-4",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 12,
    "output_tokens": 15
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Always `"message"` |
| `role` | string | Always `"assistant"` |
| `content` | array | Array of response content blocks (text, tool_use, thinking) |
| `model` | string | Model identifier used |
| `stop_reason` | string | Why generation stopped: `"end_turn"`, `"tool_use"`, `"max_tokens"` |
| `usage` | object | Token counts for prompt and completion |
| `usage.input_tokens` | number | Number of input tokens |
| `usage.output_tokens` | number | Number of output tokens |

### Citation Types

Response content may include citations with these location types:

- `char_location`: Character-level citation
- `page_location`: Page-level citation (for documents)
- `content_block_location`: Content block reference
- `web_search_result_location`: Web search result reference

## Output Configuration

Optionally specify effort level for response generation:

- `low`
- `medium`
- `high`
- `max`

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Successful response |
| 400 | Invalid request error - malformed request parameters |
| 401 | Authentication error - invalid or missing API key |
| 403 | Permission denied error |
| 404 | Not found error - model not found |
| 429 | Rate limit error - too many requests |
| 500 | API server error |
| 503 | Service overloaded - temporary capacity issues |

## Related Documentation

- [API Reference Overview](/docs/api/reference/overview) - Complete API documentation
- [Anthropic Agent SDK](/docs/guides/community/anthropic-agent-sdk) - Agent SDK integration
- [Claude Code Integration](/docs/guides/guides/claude-code-integration) - Using with Claude Code
