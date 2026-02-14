# API Error Handling and Debugging

## Error Response Structure

OpenRouter returns errors as JSON with a consistent format:

```typescript
type ErrorResponse = {
  error: {
    code: number;
    message: string;
    metadata?: Record<string, unknown>;
  };
};
```

The HTTP status code matches the `error.code` value for invalid requests or insufficient credits. Otherwise, responses return 200 OK with errors in the response body or SSE events.

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 400 | Bad Request (invalid parameters or CORS issues) |
| 401 | Invalid credentials (expired OAuth, disabled API key) |
| 402 | Insufficient credits on account or API key |
| 403 | Input flagged by required moderation system |
| 408 | Request timeout |
| 429 | Rate limit exceeded |
| 502 | Model provider unavailable or returned invalid response |
| 503 | No available provider matching routing requirements |

## Moderation Error Details

When flagged, `error.metadata` contains:

```typescript
type ModerationErrorMetadata = {
  reasons: string[];
  flagged_input: string; // Limited to 100 characters
  provider_name: string;
  model_slug: string;
};
```

## Provider Error Details

When upstream providers fail, `error.metadata` includes:

```typescript
type ProviderErrorMetadata = {
  provider_name: string;
  raw: unknown; // Original provider error
};
```

## No Content Generated

Models occasionally produce no output during warm-up periods or scaling events. Retry mechanisms or alternative providers are recommended. Note: you may still incur prompt processing charges.

## Streaming Errors

### Pre-Stream Errors
Standard error format with appropriate HTTP status codes.

### Mid-Stream Errors
Sent as Server-Sent Events with unified structure:

```typescript
type MidStreamError = {
  error: {
    code: string | number;
    message: string;
  };
  choices: [{
    finish_reason: 'error';
    native_finish_reason?: string;
  }];
};
```

## OpenAI Responses API Errors

Special event types for this endpoint:

- **`response.failed`** - Official failure event
- **`response.error`** - Error during generation
- **`error`** - Plain error event

Certain codes transform to successful completions with `length` finish reason: `context_length_exceeded`, `max_tokens_exceeded`, `token_limit_exceeded`, `string_too_long`

## Debug Mode

Enable inspection of transformed request bodies sent to providers:

```typescript
type DebugOptions = {
  echo_upstream_body?: boolean;
};
```

Include in your request:

```typescript
{
  model: 'anthropic/claude-haiku-4.5',
  stream: true,
  debug: {
    echo_upstream_body: true,
  }
}
```

Debug output appears as the first streaming chunk with empty choices array. **Limitations**: Only works with streaming chat completions, not production-recommended, and may expose sensitive request data.

### Debug Use Cases

- Understand parameter transformations across providers
- Verify message formatting and defaults
- Check provider fallback attempts
