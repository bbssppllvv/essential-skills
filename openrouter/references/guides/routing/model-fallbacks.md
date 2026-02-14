# Model Fallbacks

## Overview

OpenRouter's model fallbacks feature enables automatic switching between AI models when primary providers experience issues. The `models` parameter accepts an array of model IDs in priority order. If the first model returns an error, OpenRouter will automatically try the next model in the list.

## Fallback Triggers

Supported error types that trigger fallback include:

- Rate-limiting constraints
- Provider downtime
- Content moderation flags on filtered models
- Context length validation failures

## Pricing

Requests are priced using the model that was ultimately used, which will be returned in the `model` attribute of the response body.

## Implementation Examples

### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  models: ['anthropic/claude-3.5-sonnet', 'gryphe/mythomax-l2-13b'],
  messages: [
    {
      role: 'user',
      content: 'What is the meaning of life?',
    },
  ],
});

console.log(completion.choices[0].message.content);
```

### Fetch API

```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    models: ['anthropic/claude-3.5-sonnet', 'gryphe/mythomax-l2-13b'],
    messages: [
      {
        role: 'user',
        content: 'What is the meaning of life?',
      },
    ],
  }),
});

const data = await response.json();
console.log(data.choices[0].message.content);
```

### Python

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
    "models": ["anthropic/claude-3.5-sonnet", "gryphe/mythomax-l2-13b"],
    "messages": [
      {
        "role": "user",
        "content": "What is the meaning of life?"
      }
    ]
  })
)

data = response.json()
print(data['choices'][0]['message']['content'])
```

## OpenAI SDK Integration

### Python (OpenAI SDK)

```python
from openai import OpenAI

openai_client = OpenAI(
  base_url="https://openrouter.ai/api/v1",
  api_key="<OPENROUTER_API_KEY>",
)

completion = openai_client.chat.completions.create(
    model="openai/gpt-4o",
    extra_body={
        "models": ["anthropic/claude-3.5-sonnet", "gryphe/mythomax-l2-13b"],
    },
    messages=[
        {
            "role": "user",
            "content": "What is the meaning of life?"
        }
    ]
)

print(completion.choices[0].message.content)
```

### TypeScript (OpenAI SDK)

```typescript
import OpenAI from 'openai';

const openrouterClient = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: '<OPENROUTER_API_KEY>',
});

async function main() {
  // @ts-expect-error
  const completion = await openrouterClient.chat.completions.create({
    model: 'openai/gpt-4o',
    models: ['anthropic/claude-3.5-sonnet', 'gryphe/mythomax-l2-13b'],
    messages: [
      {
        role: 'user',
        content: 'What is the meaning of life?',
      },
    ],
  });
  console.log(completion.choices[0].message);
}

main();
```
