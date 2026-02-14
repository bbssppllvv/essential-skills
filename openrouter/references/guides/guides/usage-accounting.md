# Usage Accounting

Track AI Model Usage with OpenRouter

---

OpenRouter provides automatic usage tracking without requiring additional API calls. This feature provides detailed information about token counts, costs, and caching status directly in your API responses.

## What's Included

Every API response includes a `usage` object containing:

- Prompt and completion token counts (using the model's native tokenizer)
- Cost information in credits
- Reasoning token counts (when applicable)
- Cached token details

> **Important**: The previously deprecated parameters `usage: { include: true }` and `stream_options: { include_usage: true }` no longer have any effect. Full usage details are now always included automatically in every response.

## Usage Response Structure

For streaming responses, usage information is included in the last SSE message. For non-streaming requests, it is included in the complete response.

```json
{
  "object": "chat.completion.chunk",
  "usage": {
    "completion_tokens": 2,
    "completion_tokens_details": {
      "reasoning_tokens": 0
    },
    "cost": 0.95,
    "cost_details": {
      "upstream_inference_cost": 19
    },
    "prompt_tokens": 194,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "cache_write_tokens": 100,
      "audio_tokens": 0
    },
    "total_tokens": 196
  }
}
```

### Usage Fields

| Field | Type | Description |
|-------|------|-------------|
| `completion_tokens` | number | Number of tokens in the completion |
| `completion_tokens_details.reasoning_tokens` | number | Number of reasoning tokens (when applicable) |
| `cost` | number | Total account charge in credits |
| `cost_details.upstream_inference_cost` | number | Upstream provider cost (BYOK only) |
| `prompt_tokens` | number | Number of tokens in the prompt |
| `prompt_tokens_details.cached_tokens` | number | Number of tokens read from the cache |
| `prompt_tokens_details.cache_write_tokens` | number | Number of tokens written to the cache |
| `prompt_tokens_details.audio_tokens` | number | Number of audio tokens |
| `total_tokens` | number | Total token count (prompt + completion) |

> **Note**: `cached_tokens` represents tokens that were read from the cache. `cache_write_tokens` represents tokens that were written to the cache.

## Code Examples

### Non-Streaming Requests

#### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

const response = await openRouter.chat.send({
  model: '{{MODEL}}',
  messages: [
    {
      role: 'user',
      content: 'What is the capital of France?',
    },
  ],
});

console.log('Response:', response.choices[0].message.content);
console.log('Usage Stats:', response.usage);
```

#### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="{{API_KEY_REF}}",
)

response = client.chat.completions.create(
    model="{{MODEL}}",
    messages=[
        {"role": "user", "content": "What is the capital of France?"}
    ]
)

print("Response:", response.choices[0].message.content)
print("Usage Stats:", response.usage)
```

#### TypeScript (OpenAI SDK)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: '{{API_KEY_REF}}',
});

async function getResponseWithUsage() {
  const response = await openai.chat.completions.create({
    model: '{{MODEL}}',
    messages: [
      {
        role: 'user',
        content: 'What is the capital of France?',
      },
    ],
  });

  console.log('Response:', response.choices[0].message.content);
  console.log('Usage Stats:', response.usage);
}

getResponseWithUsage();
```

### Streaming Requests

#### Python (Streaming)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="{{API_KEY_REF}}",
)

def chat_completion_streaming(messages):
    response = client.chat.completions.create(
        model="{{MODEL}}",
        messages=messages,
        stream=True
    )
    return response

for chunk in chat_completion_streaming([
    {"role": "user", "content": "Write a haiku about Paris."}
]):
    if hasattr(chunk, 'usage') and chunk.usage:
        if hasattr(chunk.usage, 'total_tokens'):
            print(f"\nUsage Statistics:")
            print(f"Total Tokens: {chunk.usage.total_tokens}")
            print(f"Prompt Tokens: {chunk.usage.prompt_tokens}")
            print(f"Completion Tokens: {chunk.usage.completion_tokens}")
            print(f"Cost: {chunk.usage.cost} credits")
    elif chunk.choices and chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

#### TypeScript (Streaming)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: '{{API_KEY_REF}}',
});

async function chatCompletionStreaming(messages) {
  const response = await openai.chat.completions.create({
    model: '{{MODEL}}',
    messages,
    stream: true,
  });

  return response;
}

(async () => {
  for await (const chunk of chatCompletionStreaming([
    { role: 'user', content: 'Write a haiku about Paris.' },
  ])) {
    if (chunk.usage) {
      console.log('\nUsage Statistics:');
      console.log(`Total Tokens: ${chunk.usage.total_tokens}`);
      console.log(`Prompt Tokens: ${chunk.usage.prompt_tokens}`);
      console.log(`Completion Tokens: ${chunk.usage.completion_tokens}`);
      console.log(`Cost: ${chunk.usage.cost} credits`);
    } else if (chunk.choices[0]?.delta?.content) {
      process.stdout.write(chunk.choices[0].delta.content);
    }
  }
})();
```

## Asynchronous Usage Retrieval

You can also retrieve usage information asynchronously by using the generation ID returned from your API calls. This is particularly useful when you want to fetch usage statistics after the completion has finished or when you need to audit historical usage.

The generation ID is returned in the response headers or body, and you can query the `/generation` endpoint to retrieve detailed usage information for that specific request.

## Best Practices

- Monitor token consumption during development to optimize costs before moving to production
- Leverage cached token data to understand and improve application performance
- Use the cost field to track spending in real-time
- For streaming responses, check the final chunk for usage information

## Related Documentation

- [Activity Export](/docs/guides/guides/activity-export) - Export detailed usage reports
- [User Tracking](/docs/guides/guides/user-tracking) - Track usage per user
- [App Attribution](/docs/app-attribution) - Attribution and analytics
