# Rate Limits

Rate limits and credit-based quotas for the OpenRouter API

## Overview

OpenRouter implements rate limiting and credit-based quotas to manage API usage. Creating multiple accounts or API keys won't bypass these limits, as they're governed globally. However, different models have varying rate limits, allowing you to distribute load across them.

## Checking Rate Limits and Credits

To retrieve your API key's rate limit and remaining credits, submit a GET request to `https://openrouter.ai/api/v1/key`.

### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

const keyInfo = await openRouter.apiKeys.getCurrent();
console.log(keyInfo);
```

### Python

```python
import requests
import json

response = requests.get(
  url="https://openrouter.ai/api/v1/key",
  headers={
    "Authorization": f"Bearer {{API_KEY_REF}}"
  }
)

print(json.dumps(response.json(), indent=2))
```

### TypeScript (Raw API)

```typescript
const response = await fetch('https://openrouter.ai/api/v1/key', {
  method: 'GET',
  headers: {
    Authorization: 'Bearer {{API_KEY_REF}}',
  },
});

const keyInfo = await response.json();
console.log(keyInfo);
```

## Response Format

```typescript
type Key = {
  data: {
    label: string;
    limit: number | null;
    limit_reset: string | null;
    limit_remaining: number | null;
    include_byok_in_limit: boolean;

    usage: number;
    usage_daily: number;
    usage_weekly: number;
    usage_monthly: number;

    byok_usage: number;
    byok_usage_daily: number;
    byok_usage_weekly: number;
    byok_usage_monthly: number;

    is_free_tier: boolean;
  };
};
```

## Rate Limit Types

### Free Model Limits

Models with a free model variant (with an ID ending in `:free`) are restricted to a defined number of requests per minute, with daily caps depending on whether the user has purchased credits.

### DDoS Protection

Cloudflare's security measures defend against requests that dramatically exceed reasonable usage patterns.

## Payment Requirements

Accounts with negative credit balances receive payment-required errors and cannot access services until the balance becomes positive.
