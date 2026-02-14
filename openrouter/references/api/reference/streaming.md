# Streaming

Real-time Model Responses in OpenRouter

The OpenRouter API enables streaming responses from any model, making it ideal for chat interfaces where the UI updates as the model generates content.

## Enabling Streaming

To activate streaming, set the `stream` parameter to `true` in your request. The model will then send response chunks to the client progressively rather than returning everything at once.

## Implementation Examples

**TypeScript SDK:**
```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

const question = 'How would you build the tallest building ever?';

const stream = await openRouter.chat.send({
  model: '{{MODEL}}',
  messages: [{ role: 'user', content: question }],
  stream: true,
});

for await (const chunk of stream) {
  const content = chunk.choices?.[0]?.delta?.content;
  if (content) {
    console.log(content);
  }

  // Final chunk includes usage stats
  if (chunk.usage) {
    console.log('Usage:', chunk.usage);
  }
}
```

**Python:**
```python
import requests
import json

question = "How would you build the tallest building ever?"

url = "https://openrouter.ai/api/v1/chat/completions"
headers = {
  "Authorization": f"Bearer {{API_KEY_REF}}",
  "Content-Type": "application/json"
}

payload = {
  "model": "{{MODEL}}",
  "messages": [{"role": "user", "content": question}],
  "stream": True
}

buffer = ""
with requests.post(url, headers=headers, json=payload, stream=True) as r:
  for chunk in r.iter_content(chunk_size=1024, decode_unicode=True):
    buffer += chunk
    while True:
      try:
        # Find the next complete SSE line
        line_end = buffer.find('\n')
        if line_end == -1:
          break

        line = buffer[:line_end].strip()
        buffer = buffer[line_end + 1:]

        if line.startswith('data: '):
          data = line[6:]
          if data == '[DONE]':
            break

          try:
            data_obj = json.loads(data)
            content = data_obj["choices"][0]["delta"].get("content")
            if content:
              print(content, end="", flush=True)
          except json.JSONDecodeError:
            pass
      except Exception:
        break
```

**TypeScript (fetch):**
```typescript
const question = 'How would you build the tallest building ever?';
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${API_KEY_REF}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: '{{MODEL}}',
    messages: [{ role: 'user', content: question }],
    stream: true,
  }),
});

const reader = response.body?.getReader();
if (!reader) {
  throw new Error('Response body is not readable');
}

const decoder = new TextDecoder();
let buffer = '';

try {
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    // Append new chunk to buffer
    buffer += decoder.decode(value, { stream: true });

    // Process complete lines from buffer
    while (true) {
      const lineEnd = buffer.indexOf('\n');
      if (lineEnd === -1) break;

      const line = buffer.slice(0, lineEnd).trim();
      buffer = buffer.slice(lineEnd + 1);

      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') break;

        try {
          const parsed = JSON.parse(data);
          const content = parsed.choices[0].delta.content;
          if (content) {
            console.log(content);
          }
        } catch (e) {
          // Ignore invalid JSON
        }
      }
    }
  }
} finally {
  reader.cancel();
}
```

## SSE Comments

OpenRouter occasionally sends SSE comments like `: OPENROUTER PROCESSING` to prevent connection timeouts. These can be safely ignored per SSE specifications but may enhance UX with dynamic loading indicators.

Recommended SSE client libraries:
- eventsource-parser
- OpenAI SDK
- Vercel AI SDK

## Stream Cancellation

Streaming requests can be cancelled by aborting the connection. For supported providers, this stops model processing and billing immediately.

### Supported Providers

OpenAI, Azure, Anthropic, Fireworks, Mancer, Recursal, AnyScale, Lepton, OctoAI, Novita, DeepInfra, Together, Cohere, Hyperbolic, Infermatic, Avian, XAI, Cloudflare, SFCompute, Nineteen, Liquid, Friendli, Chutes, DeepSeek

### Unsupported Providers

AWS Bedrock, Groq, Modal, Google, Google AI Studio, Minimax, HuggingFace, Replicate, Perplexity, Mistral, AI21, Featherless, Lynn, Lambda, Reflection, SambaNova, Inflection, ZeroOneAI, AionLabs, Alibaba, Nebius, Kluster, Targon, InferenceNet

### TypeScript SDK Cancellation

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

const controller = new AbortController();

try {
  const stream = await openRouter.chat.send({
    model: '{{MODEL}}',
    messages: [{ role: 'user', content: 'Write a story' }],
    stream: true,
  }, {
    signal: controller.signal,
  });

  for await (const chunk of stream) {
    const content = chunk.choices?.[0]?.delta?.content;
    if (content) {
      console.log(content);
    }
  }
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Stream cancelled');
  } else {
    throw error;
  }
}

// To cancel the stream:
controller.abort();
```

### Python Cancellation

```python
import requests
from threading import Event, Thread

def stream_with_cancellation(prompt: str, cancel_event: Event):
    with requests.Session() as session:
        response = session.post(
            "https://openrouter.ai/api/v1/chat/completions",
            headers={"Authorization": f"Bearer {{API_KEY_REF}}"},
            json={"model": "{{MODEL}}", "messages": [{"role": "user", "content": prompt}], "stream": True},
            stream=True
        )

        try:
            for line in response.iter_lines():
                if cancel_event.is_set():
                    response.close()
                    return
                if line:
                    print(line.decode(), end="", flush=True)
        finally:
            response.close()

# Example usage:
cancel_event = Event()
stream_thread = Thread(target=lambda: stream_with_cancellation("Write a story", cancel_event))
stream_thread.start()

# To cancel the stream:
cancel_event.set()
```

### TypeScript (fetch) Cancellation

```typescript
const controller = new AbortController();

try {
  const response = await fetch(
    'https://openrouter.ai/api/v1/chat/completions',
    {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${API_KEY_REF}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: '{{MODEL}}',
        messages: [{ role: 'user', content: 'Write a story' }],
        stream: true,
      }),
      signal: controller.signal,
    },
  );

  // Process the stream...
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Stream cancelled');
  } else {
    throw error;
  }
}

// To cancel the stream:
controller.abort();
```

> **Warning**: Cancellation functions only for streaming with supported providers. Non-streaming requests or unsupported providers will continue processing and incur full billing.

## Error Handling During Streaming

### Pre-Stream Errors

If an error occurs before any tokens have been streamed to the client, OpenRouter returns a standard JSON error response with the appropriate HTTP status code:

```json
{
  "error": {
    "code": 400,
    "message": "Invalid model specified"
  }
}
```

Common HTTP status codes:
- **400**: Bad Request (invalid parameters)
- **401**: Unauthorized (invalid API key)
- **402**: Payment Required (insufficient credits)
- **429**: Too Many Requests (rate limited)
- **502**: Bad Gateway (provider error)
- **503**: Service Unavailable (no available providers)

### Mid-Stream Errors

If an error occurs after some tokens have already been streamed to the client, OpenRouter cannot change the HTTP status code (which is already 200 OK). Instead, the error is sent as an SSE with unified structure:

```text
data: {"id":"cmpl-abc123","object":"chat.completion.chunk","created":1234567890,"model":"gpt-3.5-turbo","provider":"openai","error":{"code":"server_error","message":"Provider disconnected unexpectedly"},"choices":[{"index":0,"delta":{"content":""},"finish_reason":"error"}]}
```

Mid-stream error characteristics:
- Error appears at top level alongside standard response fields
- A `choices` array includes `finish_reason: "error"` to terminate the stream
- HTTP status remains 200 OK since headers were already sent
- The stream terminates after the error event

### Error Handling Examples

**TypeScript SDK:**
```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

async function streamWithErrorHandling(prompt: string) {
  try {
    const stream = await openRouter.chat.send({
      model: '{{MODEL}}',
      messages: [{ role: 'user', content: prompt }],
      stream: true,
    });

    for await (const chunk of stream) {
      // Check for errors in chunk
      if ('error' in chunk) {
        console.error(`Stream error: ${chunk.error.message}`);
        if (chunk.choices?.[0]?.finish_reason === 'error') {
          console.log('Stream terminated due to error');
        }
        return;
      }

      // Process normal content
      const content = chunk.choices?.[0]?.delta?.content;
      if (content) {
        console.log(content);
      }
    }
  } catch (error) {
    // Handle pre-stream errors
    console.error(`Error: ${error.message}`);
  }
}
```

**Python:**
```python
import requests
import json

async def stream_with_error_handling(prompt):
    response = requests.post(
        'https://openrouter.ai/api/v1/chat/completions',
        headers={'Authorization': f'Bearer {{API_KEY_REF}}'},
        json={
            'model': '{{MODEL}}',
            'messages': [{'role': 'user', 'content': prompt}],
            'stream': True
        },
        stream=True
    )

    # Check initial HTTP status for pre-stream errors
    if response.status_code != 200:
        error_data = response.json()
        print(f"Error: {error_data['error']['message']}")
        return

    # Process stream and handle mid-stream errors
    for line in response.iter_lines():
        if line:
            line_text = line.decode('utf-8')
            if line_text.startswith('data: '):
                data = line_text[6:]
                if data == '[DONE]':
                    break

                try:
                    parsed = json.loads(data)

                    # Check for mid-stream error
                    if 'error' in parsed:
                        print(f"Stream error: {parsed['error']['message']}")
                        # Check finish_reason if needed
                        if parsed.get('choices', [{}])[0].get('finish_reason') == 'error':
                            print("Stream terminated due to error")
                        break

                    # Process normal content
                    content = parsed['choices'][0]['delta'].get('content')
                    if content:
                        print(content, end='', flush=True)

                except json.JSONDecodeError:
                    pass
```

**TypeScript (fetch):**
```typescript
async function streamWithErrorHandling(prompt: string) {
  const response = await fetch(
    'https://openrouter.ai/api/v1/chat/completions',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${API_KEY_REF}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: '{{MODEL}}',
        messages: [{ role: 'user', content: prompt }],
        stream: true,
      }),
    }
  );

  // Check initial HTTP status for pre-stream errors
  if (!response.ok) {
    const error = await response.json();
    console.error(`Error: ${error.error.message}`);
    return;
  }

  const reader = response.body?.getReader();
  if (!reader) throw new Error('No response body');

  const decoder = new TextDecoder();
  let buffer = '';

  try {
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      buffer += decoder.decode(value, { stream: true });

      while (true) {
        const lineEnd = buffer.indexOf('\n');
        if (lineEnd === -1) break;

        const line = buffer.slice(0, lineEnd).trim();
        buffer = buffer.slice(lineEnd + 1);

        if (line.startsWith('data: ')) {
          const data = line.slice(6);
          if (data === '[DONE]') return;

          try {
            const parsed = JSON.parse(data);

            // Check for mid-stream error
            if (parsed.error) {
              console.error(`Stream error: ${parsed.error.message}`);
              // Check finish_reason if needed
              if (parsed.choices?.[0]?.finish_reason === 'error') {
                console.log('Stream terminated due to error');
              }
              return;
            }

            // Process normal content
            const content = parsed.choices[0].delta.content;
            if (content) {
              console.log(content);
            }
          } catch (e) {
            // Ignore parsing errors
          }
        }
      }
    }
  } finally {
    reader.cancel();
  }
}
```

### API-Specific Behavior

Different API endpoints handle streaming errors variably:
- **OpenAI Chat Completions API**: Returns error responses directly if no chunks processed, or includes error information in response if chunks were processed
- **OpenAI Responses API**: May transform certain error codes (like `context_length_exceeded`) into successful responses with `finish_reason: "length"` rather than treating them as errors
