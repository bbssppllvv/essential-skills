# API Rate Limits Documentation

## Overview
OpenRouter implements rate limits and credit-based quotas to manage API usage. Creating multiple accounts or API keys won't bypass these limits, as they're governed globally. However, different models have varying rate limits, allowing you to distribute load across them.

## Checking Rate Limits and Credits

To retrieve your API key's rate limit and remaining credits, submit a GET request to `https://openrouter.ai/api/v1/key`.

### Code Examples

**TypeScript SDK:**
```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '{{API_KEY_REF}}',
});

const keyInfo = await openRouter.apiKeys.getCurrent();
console.log(keyInfo);
```

**Python:**
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

**TypeScript (Raw API):**
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

## Response Structure

The API returns key information including credit limits, remaining credits, and usage metrics across different time periods (daily, weekly, monthly).

## Rate Limit Types

1. **Free Model Limits**: Models with IDs ending in specific variants are restricted to a defined number of requests per minute and daily request quotas, varying based on whether users have purchased credits.

2. **DDoS Protection**: Cloudflare's security measures block requests exhibiting unusual patterns.

## Payment Requirements

Accounts with negative credit balances receive payment-required errors and cannot access services until the balance becomes positive.
