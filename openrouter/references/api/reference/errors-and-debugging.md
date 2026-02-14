# Errors and Debugging

Error handling and debugging for the OpenRouter API

## Error Response Structure

OpenRouter returns errors as JSON with this format:

```typescript
type ErrorResponse = {
  error: {
    code: number;
    message: string;
    metadata?: Record<string, unknown>;
  };
};
```

The HTTP status code matches the `error.code` value when requests are invalid or credits are insufficient. Otherwise, responses return 200 OK with errors embedded in the body or SSE events.

## Standard Error Codes

| Code | Meaning |
|------|---------|
| 400 | Bad Request (invalid params, CORS issues) |
| 401 | Invalid credentials or expired OAuth |
| 402 | Insufficient account credits |
| 403 | Input flagged by required moderation |
| 408 | Request timeout |
| 429 | Rate limit exceeded |
| 502 | Model provider unavailable or returned invalid response |
| 503 | No provider available matching routing requirements |

## Moderation Error Metadata

When inputs are flagged, metadata includes:

```typescript
type ModerationErrorMetadata = {
  reasons: string[];
  flagged_input: string;
  provider_name: string;
  model_slug: string;
};
```

The flagged input is limited to 100 characters. If the flagged input is longer than 100 characters, it will be truncated in the middle and replaced with `...`.

## Provider Error Metadata

```typescript
type ProviderErrorMetadata = {
  provider_name: string;
  raw: unknown;
};
```

## No Content Generation

Models occasionally generate no content due to warming up from a cold start, or because the system is scaling up to handle more requests. Warm-up times usually range from a few seconds to a few minutes. Users may still be charged for prompt processing even without generated content.

Retry mechanisms or alternative providers are recommended.

## Streaming Error Handling

### Pre-Stream Errors

Standard error format with appropriate HTTP status codes.

### Mid-Stream Errors

Server-Sent Events with unified structure:

```typescript
type MidStreamError = {
  id: string;
  object: 'chat.completion.chunk';
  created: number;
  model: string;
  provider: string;
  error: {
    code: string | number;
    message: string;
  };
  choices: [{
    index: 0;
    delta: { content: '' };
    finish_reason: 'error';
    native_finish_reason?: string;
  }];
};
```

Example SSE event:

```text
data: {"id":"cmpl-abc123","object":"chat.completion.chunk","created":1234567890,"model":"gpt-3.5-turbo","provider":"openai","error":{"code":"server_error","message":"Provider disconnected"},"choices":[{"index":0,"delta":{"content":""},"finish_reason":"error"}]}
```

## OpenAI Responses API Error Events

The Responses API (`/api/alpha/responses`) uses specific event types:

**`response.failed`** - Official failure event:
```json
{
  "type": "response.failed",
  "response": {
    "id": "resp_abc123",
    "status": "failed",
    "error": {
      "code": "server_error",
      "message": "Internal server error"
    }
  }
}
```

**`response.error`** - Error during generation:
```json
{
  "type": "response.error",
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded"
  }
}
```

**`error`** - Plain error event:
```json
{
  "type": "error",
  "error": {
    "code": "invalid_api_key",
    "message": "Invalid API key provided"
  }
}
```

### Error Code Transformations

Certain errors become successful responses:

| Error Code | Transformed To | Finish Reason |
|-----------|----------------|---------------|
| `context_length_exceeded` | Success | `length` |
| `max_tokens_exceeded` | Success | `length` |
| `token_limit_exceeded` | Success | `length` |
| `string_too_long` | Success | `length` |

## Debugging with Debug Option

The `debug` parameter enables inspection of transformed requests:

```typescript
type DebugOptions = {
  echo_upstream_body?: boolean;
};
```

### TypeScript Example

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'anthropic/claude-haiku-4.5',
    stream: true,
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' },
      { role: 'user', content: 'Hello!' },
    ],
    debug: {
      echo_upstream_body: true,
    },
  }),
});

const text = await response.text();

for (const line of text.split('\n')) {
  if (!line.startsWith('data: ')) continue;

  const data = line.slice(6);
  if (data === '[DONE]') break;

  const parsed = JSON.parse(data);

  if (parsed.debug?.echo_upstream_body) {
    console.log('\nDebug:', JSON.stringify(parsed.debug.echo_upstream_body, null, 2));
  }

  process.stdout.write(parsed.choices?.[0]?.delta?.content ?? '');
}
```

### Python Example

```python
import requests
import json

response = requests.post(
  url="https://openrouter.ai/api/v1/chat/completions",
  headers={
    "Authorization": "Bearer <OPENROUTER_API_KEY>",
    "Content-Type": "application/json",
  },
  data=json.dumps({
    "model": "anthropic/claude-haiku-4.5",
    "stream": True,
    "messages": [
      { "role": "system", "content": "You are a helpful assistant." },
      { "role": "user", "content": "Hello!" }
    ],
    "debug": {
      "echo_upstream_body": True
    }
  }),
  stream=True
)

for line in response.iter_lines():
  if line:
    text = line.decode('utf-8')
    if 'echo_upstream_body' in text:
      print(text)
```

### Debug Response Format

The first chunk in streaming responses includes debug data:

```json
{
  "id": "gen-xxxxx",
  "provider": "Anthropic",
  "model": "anthropic/claude-haiku-4.5",
  "object": "chat.completion.chunk",
  "created": 1234567890,
  "choices": [],
  "debug": {
    "echo_upstream_body": {
      "system": [
        { "type": "text", "text": "You are a helpful assistant." }
      ],
      "messages": [
        { "role": "user", "content": "Hello!" }
      ],
      "model": "claude-haiku-4-5-20251001",
      "stream": true,
      "max_tokens": 64000,
      "temperature": 1
    }
  }
}
```

## Debug Limitations

The debug option only works with streaming mode (`stream: true`) for the Chat Completions API. It should not be used in production environments as it may expose sensitive information.

## Debug Use Cases

1. Understanding how parameters are transformed across providers
2. Verifying message formatting
3. Checking applied defaults
4. Debugging provider fallback attempts
