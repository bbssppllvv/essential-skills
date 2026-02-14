# Authentication

API key authentication for OpenRouter

## Overview

OpenRouter uses Bearer token authentication for API requests, compatible with standard tools like `curl` and the OpenAI SDK. API keys on OpenRouter are more powerful than keys used directly for model APIs, since they support credit limits and OAuth flows.

## Key Setup

Create an API key at [openrouter.ai/keys](https://openrouter.ai/keys), optionally setting spending restrictions. The Authorization header should be formatted as `Bearer <OPENROUTER_API_KEY>` for direct API calls.

## Implementation Examples

**TypeScript with OpenRouter SDK:**
```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
  defaultHeaders: {
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
  },
});

const completion = await openRouter.chat.send({
  model: 'openai/gpt-5.2',
  messages: [{ role: 'user', content: 'Say this is a test' }],
  stream: false,
});

console.log(completion.choices[0].message);
```

**Python with OpenAI SDK:**
```python
from openai import OpenAI

client = OpenAI(
  base_url="https://openrouter.ai/api/v1",
  api_key="<OPENROUTER_API_KEY>",
)

response = client.chat.completions.create(
  extra_headers={
    "HTTP-Referer": "<YOUR_SITE_URL>",
    "X-Title": "<YOUR_SITE_NAME>",
  },
  model="openai/gpt-5.2",
  messages=[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ],
)

reply = response.choices[0].message
```

**TypeScript with OpenAI SDK:**
```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: '<OPENROUTER_API_KEY>',
  defaultHeaders: {
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
  },
});

async function main() {
  const completion = await openai.chat.completions.create({
    model: 'openai/gpt-5.2',
    messages: [{ role: 'user', content: 'Say this is a test' }],
  });

  console.log(completion.choices[0].message);
}

main();
```

**Raw Fetch API:**
```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      {
        role: 'user',
        content: 'What is the meaning of life?',
      },
    ],
  }),
});
```

**cURL:**
```shell
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -d '{
  "model": "openai/gpt-5.2",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ]
}'
```

## Security Considerations

You must protect your API keys and never commit them to public repositories. OpenRouter is a [GitHub secret scanning partner](https://docs.github.com/en/code-security/secret-scanning/introduction/supported-secret-scanning-patterns) and monitors for exposed keys. If a compromise is detected, you will be notified via email.

Immediate action required upon exposure: delete the compromised key and generate a replacement via the [settings page](https://openrouter.ai/keys). Environment variables are strongly recommended for key storage.
