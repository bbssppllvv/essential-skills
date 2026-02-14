# App Attribution

Get your app featured in OpenRouter rankings and analytics

---

## Benefits of App Attribution

When you properly attribute your app usage, you gain access to:

- **Public App Rankings**: Your app appears in OpenRouter's public rankings with daily, weekly, and monthly leaderboards
- **Model Apps Tabs**: Your app is featured on individual model pages showing which apps use each model most
- **Detailed Analytics**: Access comprehensive analytics showing your app's model usage over time, token consumption, and usage patterns
- **Professional Visibility**: Showcase your app to the OpenRouter developer community

## Attribution Headers

OpenRouter tracks app attribution through two optional HTTP headers:

### HTTP-Referer

The `HTTP-Referer` header identifies your app's URL and is used as the primary identifier for rankings.

### X-Title

The `X-Title` header sets or modifies your app's display name in rankings and analytics.

> **Note**: Both headers are optional, but including them enables all attribution features. Apps using localhost URLs must include a title to be tracked.

---

## Implementation Examples

### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
  defaultHeaders: {
    'HTTP-Referer': 'https://myapp.com',
    'X-Title': 'My AI Assistant',
  },
});

const completion = await openRouter.chat.send({
  model: 'openai/gpt-5.2',
  messages: [
    {
      role: 'user',
      content: 'Hello, world!',
    },
  ],
  stream: false,
});

console.log(completion.choices[0].message);
```

### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
  base_url="https://openrouter.ai/api/v1",
  api_key="<OPENROUTER_API_KEY>",
)

completion = client.chat.completions.create(
  extra_headers={
    "HTTP-Referer": "https://myapp.com",
    "X-Title": "My AI Assistant",
  },
  model="openai/gpt-5.2",
  messages=[
    {
      "role": "user",
      "content": "Hello, world!"
    }
  ]
)
```

### TypeScript (OpenAI SDK)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: '<OPENROUTER_API_KEY>',
  defaultHeaders: {
    'HTTP-Referer': 'https://myapp.com',
    'X-Title': 'My AI Assistant',
  },
});

async function main() {
  const completion = await openai.chat.completions.create({
    model: 'openai/gpt-5.2',
    messages: [
      {
        role: 'user',
        content: 'Hello, world!',
      },
    ],
  });

  console.log(completion.choices[0].message);
}

main();
```

### Python (Direct API)

```python
import requests
import json

response = requests.post(
  url="https://openrouter.ai/api/v1/chat/completions",
  headers={
    "Authorization": "Bearer <OPENROUTER_API_KEY>",
    "HTTP-Referer": "https://myapp.com",
    "X-Title": "My AI Assistant",
    "Content-Type": "application/json",
  },
  data=json.dumps({
    "model": "openai/gpt-5.2",
    "messages": [
      {
        "role": "user",
        "content": "Hello, world!"
      }
    ]
  })
)
```

### TypeScript (fetch)

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': 'https://myapp.com',
    'X-Title': 'My AI Assistant',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      {
        role: 'user',
        content: 'Hello, world!',
      },
    ],
  }),
});
```

### cURL

```shell
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "HTTP-Referer: https://myapp.com" \
  -H "X-Title: My AI Assistant" \
  -d '{
  "model": "openai/gpt-5.2",
  "messages": [
    {
      "role": "user",
      "content": "Hello, world!"
    }
  ]
}'
```

---

## Where Your App Appears

### App Rankings

Your attributed app will appear in OpenRouter's main rankings page at [openrouter.ai/rankings](https://openrouter.ai/rankings). The rankings show:

- **Top Apps**: Largest public apps by token usage
- **Time Periods**: Daily, weekly, and monthly views
- **Usage Metrics**: Total token consumption across all models

### Model Apps Tabs

On individual model pages (e.g., GPT-4o), your app will be featured in the "Apps" tab showing:

- **Top Apps**: Apps using that specific model most
- **Weekly Rankings**: Updated weekly based on usage
- **Usage Context**: How your app compares to others using the same model

### Individual App Analytics

Once your app is tracked, you can access detailed analytics at `openrouter.ai/apps?url=<your-app-url>` including:

- **Model Usage Over Time**: Charts showing which models your app uses
- **Token Consumption**: Detailed breakdown of prompt and completion tokens
- **Usage Patterns**: Historical data to understand your app's AI usage trends

---

## Best Practices

### URL Requirements

- Use your app's primary domain (e.g., `https://myapp.com`)
- Avoid using subdomains unless they represent distinct apps
- For localhost development, always include a title header

### Title Guidelines

- Keep titles concise and descriptive
- Use your app's actual name as users know it
- Avoid generic names like "AI App" or "Chatbot"

### Privacy Considerations

- Only public apps (those that send headers) are included in rankings
- Attribution headers don't expose sensitive information about your requests

---

## Related Documentation

- [Quickstart Guide](/docs/quickstart) - Basic setup with attribution headers
- [API Reference](/docs/api/reference/overview) - Complete header documentation
- [Usage Accounting](/docs/guides/guides/usage-accounting) - Understanding your API usage
