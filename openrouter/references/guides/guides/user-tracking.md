# User Tracking

Track Your Users with OpenRouter

---

OpenRouter's user tracking feature allows you to monitor your end-users through an optional `user` parameter in API requests. The parameter accepts any string identifier that represents your end-user, such as user IDs, email hashes, or session identifiers.

## Benefits

### Improved Caching

OpenRouter can make caches sticky to your individual users, improving load-balancing and throughput. A given user of your application will always get routed to the same provider and the cache will stay warm, while separate users distribute across providers for improved throughput.

### Enhanced Reporting

The `user` parameter is available in:

- The `/activity` page
- The data exports from that page
- The `/generations` API endpoint

You can view requests broken down by user ID in your OpenRouter dashboard and understand which users are making the most requests through usage analytics.

## Implementation

Simply include a `user` parameter in your API requests with any string identifier that represents your end-user.

### Request Format

```json
{
  "model": "openai/gpt-4o",
  "messages": [
    {"role": "user", "content": "Hello, how are you?"}
  ],
  "user": "user_12345"
}
```

### TypeScript SDK

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
      content: "What's the weather like today?",
    },
  ],
  user: 'user_12345', // Your user identifier
  stream: false,
});

console.log(response.choices[0].message.content);
```

### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="{{API_KEY_REF}}",
)

response = client.chat.completions.create(
    model="{{MODEL}}",
    messages=[
        {"role": "user", "content": "What's the weather like today?"}
    ],
    user="user_12345",  # Your user identifier
)

print(response.choices[0].message.content)
```

### TypeScript (OpenAI SDK)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: '{{API_KEY_REF}}',
});

async function chatWithUserTracking() {
  const response = await openai.chat.completions.create({
    model: '{{MODEL}}',
    messages: [
      {
        role: 'user',
        content: "What's the weather like today?",
      },
    ],
    user: 'user_12345', // Your user identifier
  });

  console.log(response.choices[0].message.content);
}

chatWithUserTracking();
```

## Best Practices

- **Use consistent, stable identifiers** for the same user across requests. This ensures caching and analytics work properly.
- **Avoid random strings** that change between requests. Changing identifiers defeats the purpose of user tracking.
- **Do not include personally identifiable information** (PII) directly. Use internal user IDs rather than exposing personal information like email addresses or names.

## Related Documentation

- [Usage Accounting](/docs/guides/guides/usage-accounting) - Track token usage and costs
- [Activity Export](/docs/guides/guides/activity-export) - Export usage data
- [App Attribution](/docs/app-attribution) - App-level analytics
