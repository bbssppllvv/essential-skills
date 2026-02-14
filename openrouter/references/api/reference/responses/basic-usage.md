# Responses API Beta Basic Usage

## Overview
OpenRouter's Responses API Beta facilitates text generation through straightforward string inputs or structured message arrays. **Note:** This API is in beta and may undergo breaking changes.

## Simple String Input

The most basic approach uses a direct text string:

```typescript
const response = await fetch('https://openrouter.ai/api/v1/responses', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_OPENROUTER_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/o4-mini',
    input: 'What is the meaning of life?',
    max_output_tokens: 9000,
  }),
});
```

Python and cURL implementations are also available for this pattern.

## Structured Message Format

For multi-turn conversations, use array-based message structures with role and content definitions:

```typescript
input: [
  {
    type: 'message',
    role: 'user',
    content: [
      {
        type: 'input_text',
        text: 'Tell me a joke about programming',
      },
    ],
  },
]
```

## Response Structure

Responses follow a consistent format containing:
- Unique response identifier
- Generated output content with metadata
- Token usage statistics
- Completion status

## Streaming Implementation

Enable real-time response generation via the `stream: true` parameter. The server transmits Server-Sent Events (SSE) containing incremental content deltas that progressively build the complete response.

## Essential Parameters

| Parameter | Type | Purpose |
|-----------|------|---------|
| `model` | string | Required; specifies the AI model |
| `input` | string/array | Required; user prompt or message history |
| `stream` | boolean | Activates streaming mode |
| `max_output_tokens` | integer | Output length constraint |
| `temperature` | number | Controls response randomness (0-2) |
| `top_p` | number | Nucleus sampling control (0-1) |

## Multi-Turn Conversations

The API is stateless, requiring complete conversation history in every request. Assistant responses must include `id` and `status` fields when referenced in subsequent messages.

## Error Management

Implement try-catch blocks to handle network failures and HTTP errors, checking response status codes and parsing error messages accordingly.
