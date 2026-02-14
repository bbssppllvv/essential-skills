# Provider Routing

Route requests to the best provider.

OpenRouter routes requests to the best available providers for your model. By default, requests are load balanced across the top providers to maximize uptime.

You can customize how your requests are routed using the `provider` object in the request body for Chat Completions.

## Provider Object Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `order` | string[] | - | List of provider slugs to try in order (e.g. `["anthropic", "openai"]`) |
| `allow_fallbacks` | boolean | `true` | Whether to allow backup providers when primary unavailable |
| `require_parameters` | boolean | `false` | Only use providers supporting all request parameters |
| `data_collection` | "allow" \| "deny" | "allow" | Control data storage provider usage |
| `zdr` | boolean | - | Restrict to Zero Data Retention endpoints |
| `enforce_distillable_text` | boolean | - | Restrict to distillation-allowed models |
| `only` | string[] | - | List of allowed provider slugs |
| `ignore` | string[] | - | List of provider slugs to skip |
| `quantizations` | string[] | - | Quantization level filters |
| `sort` | string \| object | - | Sort by price, throughput, or latency |
| `preferred_min_throughput` | number \| object | - | Minimum throughput threshold (tokens/sec) |
| `preferred_max_latency` | number \| object | - | Maximum latency threshold (seconds) |
| `max_price` | object | - | Maximum acceptable pricing |

## Price-Based Load Balancing Strategy

1. Prioritize providers without significant outages in last 30 seconds
2. Select lowest-cost stable providers weighted by inverse square of price
3. Use remaining providers as fallbacks

**Load Balancing Example:** Provider A ($1/M tokens) is 9x more likely routed than Provider C ($3/M) because 1/(3^2) = 1/9.

## Provider Sorting

### Sort Options

- `"price"`: prioritize lowest price
- `"throughput"`: prioritize highest throughput
- `"latency"`: prioritize lowest latency

### TypeScript SDK Example - Throughput Sorting

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.3-70b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    sort: 'throughput',
  },
  stream: false,
});
```

### TypeScript (fetch) Example - Throughput Sorting

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'meta-llama/llama-3.3-70b-instruct',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      sort: 'throughput',
    },
  }),
});
```

### Python Example - Throughput Sorting

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'meta-llama/llama-3.3-70b-instruct',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'sort': 'throughput',
  },
})
```

## Nitro Shortcut

Append `:nitro` to model slug to sort by throughput (equivalent to `provider.sort: "throughput"`).

### TypeScript SDK Example - Nitro Shortcut

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.3-70b-instruct:nitro',
  messages: [{ role: 'user', content: 'Hello' }],
  stream: false,
});
```

### TypeScript (fetch) Example - Nitro Shortcut

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'meta-llama/llama-3.3-70b-instruct:nitro',
    messages: [{ role: 'user', content: 'Hello' }],
  }),
});
```

### Python Example - Nitro Shortcut

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'meta-llama/llama-3.3-70b-instruct:nitro',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
})
```

## Floor Price Shortcut

Append `:floor` to model slug to sort by price (equivalent to `provider.sort: "price"`).

### TypeScript SDK Example - Floor Shortcut

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.3-70b-instruct:floor',
  messages: [{ role: 'user', content: 'Hello' }],
  stream: false,
});
```

### TypeScript (fetch) Example - Floor Shortcut

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'meta-llama/llama-3.3-70b-instruct:floor',
    messages: [{ role: 'user', content: 'Hello' }],
  }),
});
```

### Python Example - Floor Shortcut

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'meta-llama/llama-3.3-70b-instruct:floor',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
})
```

## Advanced Sorting with Partition

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `sort.by` | string | - | Sorting strategy: `"price"`, `"throughput"`, or `"latency"` |
| `sort.partition` | string | `"model"` | Grouping: `"model"` (default) or `"none"` |

### Use Case 1: Highest Throughput or Lowest Latency Model

#### TypeScript SDK Example - Partition None Throughput

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  models: [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
    'google/gemini-3-flash-preview',
  ],
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    sort: {
      by: 'throughput',
      partition: 'none',
    },
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Partition None Throughput

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    models: [
      'anthropic/claude-sonnet-4.5',
      'openai/gpt-5-mini',
      'google/gemini-3-flash-preview',
    ],
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      sort: {
        by: 'throughput',
        partition: 'none',
      },
    },
  }),
});
```

#### Python Example - Partition None Throughput

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'models': [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
    'google/gemini-3-flash-preview',
  ],
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'sort': {
      'by': 'throughput',
      'partition': 'none',
    },
  },
})
```

#### cURL Example - Partition None Throughput

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer <OPENROUTER_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "models": [
      "anthropic/claude-sonnet-4.5",
      "openai/gpt-5-mini",
      "google/gemini-3-flash-preview"
    ],
    "messages": [{ "role": "user", "content": "Hello" }],
    "provider": {
      "sort": {
        "by": "throughput",
        "partition": "none"
      }
    }
  }'
```

## Performance Thresholds

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `preferred_min_throughput` | number \| object | - | Minimum throughput in tokens/sec |
| `preferred_max_latency` | number \| object | - | Maximum latency in seconds |

### Percentile Levels

- **p50** (median): 50% of requests perform better
- **p75**: 75% of requests perform better
- **p90**: 90% of requests perform better
- **p99**: 99% of requests perform better

**Important**: These preferences are soft constraints -- providers missing thresholds become fallbacks rather than being excluded entirely, ensuring request reliability.

### Use Case 2: Cheapest Model Meeting Performance Requirements

#### TypeScript SDK Example - Price with Throughput Threshold

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  models: [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
    'google/gemini-3-flash-preview',
  ],
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    sort: {
      by: 'price',
      partition: 'none',
    },
    preferredMinThroughput: {
      p90: 50,
    },
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Price with Throughput Threshold

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    models: [
      'anthropic/claude-sonnet-4.5',
      'openai/gpt-5-mini',
      'google/gemini-3-flash-preview',
    ],
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      sort: {
        by: 'price',
        partition: 'none',
      },
      preferred_min_throughput: {
        p90: 50,
      },
    },
  }),
});
```

#### Python Example - Price with Throughput Threshold

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'models': [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
    'google/gemini-3-flash-preview',
  ],
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'sort': {
      'by': 'price',
      'partition': 'none',
    },
    'preferred_min_throughput': {
      'p90': 50,
    },
  },
})
```

#### cURL Example - Price with Throughput Threshold

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer <OPENROUTER_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "models": [
      "anthropic/claude-sonnet-4.5",
      "openai/gpt-5-mini",
      "google/gemini-3-flash-preview"
    ],
    "messages": [{ "role": "user", "content": "Hello" }],
    "provider": {
      "sort": {
        "by": "price",
        "partition": "none"
      },
      "preferred_min_throughput": {
        "p90": 50
      }
    }
  }'
```

### Max Latency Threshold

#### TypeScript SDK Example - Price with Latency Threshold

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  models: [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
  ],
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    sort: {
      by: 'price',
      partition: 'none',
    },
    preferredMaxLatency: {
      p90: 3,
    },
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Price with Latency Threshold

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    models: [
      'anthropic/claude-sonnet-4.5',
      'openai/gpt-5-mini',
    ],
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      sort: {
        by: 'price',
        partition: 'none',
      },
      preferred_max_latency: {
        p90: 3,
      },
    },
  }),
});
```

#### Python Example - Price with Latency Threshold

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'models': [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
  ],
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'sort': {
      'by': 'price',
      'partition': 'none',
    },
    'preferred_max_latency': {
      'p90': 3,
    },
  },
})
```

#### cURL Example - Price with Latency Threshold

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer <OPENROUTER_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "models": [
      "anthropic/claude-sonnet-4.5",
      "openai/gpt-5-mini"
    ],
    "messages": [{ "role": "user", "content": "Hello" }],
    "provider": {
      "sort": {
        "by": "price",
        "partition": "none"
      },
      "preferred_max_latency": {
        "p90": 3
      }
    }
  }'
```

### Multiple Percentile Cutoffs Example

#### TypeScript SDK Example - Multiple Percentiles

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'deepseek/deepseek-v3.2',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    preferredMaxLatency: {
      p50: 1,
      p90: 3,
      p99: 5,
    },
    preferredMinThroughput: {
      p50: 100,
      p90: 50,
    },
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Multiple Percentiles

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'deepseek/deepseek-v3.2',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      preferred_max_latency: {
        p50: 1,
        p90: 3,
        p99: 5,
      },
      preferred_min_throughput: {
        p50: 100,
        p90: 50,
      },
    },
  }),
});
```

#### Python Example - Multiple Percentiles

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'deepseek/deepseek-v3.2',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'preferred_max_latency': {
      'p50': 1,
      'p90': 3,
      'p99': 5,
    },
    'preferred_min_throughput': {
      'p50': 100,
      'p90': 50,
    },
  },
})
```

#### cURL Example - Multiple Percentiles

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer <OPENROUTER_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek/deepseek-v3.2",
    "messages": [{ "role": "user", "content": "Hello" }],
    "provider": {
      "preferred_max_latency": {
        "p50": 1,
        "p90": 3,
        "p99": 5
      },
      "preferred_min_throughput": {
        "p50": 100,
        "p90": 50
      }
    }
  }'
```

### Use Case 3: Maximize BYOK Usage Across Models

#### TypeScript SDK Example - BYOK Across Models

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  models: [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
    'google/gemini-3-flash-preview',
  ],
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    sort: {
      by: 'price',
      partition: 'none',
    },
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - BYOK Across Models

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    models: [
      'anthropic/claude-sonnet-4.5',
      'openai/gpt-5-mini',
      'google/gemini-3-flash-preview',
    ],
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      sort: {
        by: 'price',
        partition: 'none',
      },
    },
  }),
});
```

#### Python Example - BYOK Across Models

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'models': [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
    'google/gemini-3-flash-preview',
  ],
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'sort': {
      'by': 'price',
      'partition': 'none',
    },
  },
})
```

#### cURL Example - BYOK Across Models

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer <OPENROUTER_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "models": [
      "anthropic/claude-sonnet-4.5",
      "openai/gpt-5-mini",
      "google/gemini-3-flash-preview"
    ],
    "messages": [{ "role": "user", "content": "Hello" }],
    "provider": {
      "sort": {
        "by": "price",
        "partition": "none"
      }
    }
  }'
```

## Ordering Specific Providers

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `order` | string[] | - | List of provider slugs to try in order |

### Example: Specifying providers with fallbacks

#### TypeScript SDK Example - Provider Ordering with Fallbacks

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'mistralai/mixtral-8x7b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    order: ['openai', 'together'],
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Provider Ordering with Fallbacks

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'mistralai/mixtral-8x7b-instruct',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      order: ['openai', 'together'],
    },
  }),
});
```

#### Python Example - Provider Ordering with Fallbacks

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'mistralai/mixtral-8x7b-instruct',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'order': ['openai', 'together'],
  },
})
```

### Example: Specifying providers with fallbacks disabled

#### TypeScript SDK Example - Provider Ordering No Fallbacks

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'mistralai/mixtral-8x7b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    order: ['openai', 'together'],
    allowFallbacks: false,
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Provider Ordering No Fallbacks

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'mistralai/mixtral-8x7b-instruct',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      order: ['openai', 'together'],
      allow_fallbacks: false,
    },
  }),
});
```

#### Python Example - Provider Ordering No Fallbacks

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'mistralai/mixtral-8x7b-instruct',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'order': ['openai', 'together'],
    'allow_fallbacks': False,
  },
})
```

## Targeting Specific Provider Endpoints

#### TypeScript SDK Example - DeepSeek Turbo Endpoint

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'deepseek/deepseek-r1',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    order: ['deepinfra/turbo'],
    allowFallbacks: false,
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - DeepSeek Turbo Endpoint

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'deepseek/deepseek-r1',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      order: ['deepinfra/turbo'],
      allow_fallbacks: false,
    },
  }),
});
```

#### Python Example - DeepSeek Turbo Endpoint

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'deepseek/deepseek-r1',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'order': ['deepinfra/turbo'],
    'allow_fallbacks': False,
  },
})
```

## Requiring Providers to Support All Parameters

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `require_parameters` | boolean | `false` | Only use providers supporting all parameters |

### Example: Excluding providers that don't support JSON formatting

#### TypeScript SDK Example - Require JSON Support

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    requireParameters: true,
  },
  responseFormat: { type: 'json_object' },
  stream: false,
});
```

#### TypeScript (fetch) Example - Require JSON Support

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      require_parameters: true,
    },
    response_format: { type: 'json_object' },
  }),
});
```

#### Python Example - Require JSON Support

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'require_parameters': True,
  },
  'response_format': { 'type': 'json_object' },
})
```

## Requiring Providers to Comply with Data Policies

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `data_collection` | "allow" \| "deny" | "allow" | Control data storage provider usage |

- `allow`: (default) allow providers storing user data
- `deny`: use only providers not collecting user data

### Example: Excluding providers that don't comply with data policies

#### TypeScript SDK Example - Deny Data Collection

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    dataCollection: 'deny',
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Deny Data Collection

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      data_collection: 'deny',
    },
  }),
});
```

#### Python Example - Deny Data Collection

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'data_collection': 'deny',
  },
})
```

## Zero Data Retention Enforcement

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `zdr` | boolean | - | Restrict to Zero Data Retention endpoints |

### Example: Enforcing ZDR for a specific request

#### TypeScript SDK Example - ZDR Enforcement

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'gpt-4',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    zdr: true,
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - ZDR Enforcement

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'gpt-4',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      zdr: true,
    },
  }),
});
```

#### Python Example - ZDR Enforcement

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'gpt-4',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'zdr': True,
  },
})
```

## Distillable Text Enforcement

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enforce_distillable_text` | boolean | - | Restrict to distillation-allowed models |

### Example: Enforcing distillable text for a specific request

#### TypeScript SDK Example - Distillable Text Enforcement

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.3-70b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    enforceDistillableText: true,
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Distillable Text Enforcement

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'meta-llama/llama-3.3-70b-instruct',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      enforce_distillable_text: true,
    },
  }),
});
```

#### Python Example - Distillable Text Enforcement

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'meta-llama/llama-3.3-70b-instruct',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'enforce_distillable_text': True,
  },
})
```

## Disabling Fallbacks

#### TypeScript SDK Example - Disable Fallbacks

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    allowFallbacks: false,
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Disable Fallbacks

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      allow_fallbacks: false,
    },
  }),
});
```

#### Python Example - Disable Fallbacks

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'allow_fallbacks': False,
  },
})
```

## Allowing Only Specific Providers

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `only` | string[] | - | List of allowed provider slugs |

### Example: Allowing Azure for a request calling GPT-4 Omni

#### TypeScript SDK Example - Azure Only

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'openai/gpt-5-mini',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    only: ['azure'],
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Azure Only

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5-mini',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      only: ['azure'],
    },
  }),
});
```

#### Python Example - Azure Only

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'openai/gpt-5-mini',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'only': ['azure'],
  },
})
```

## Ignoring Providers

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ignore` | string[] | - | List of provider slugs to skip |

### Example: Ignoring DeepInfra for a request calling Llama 3.3 70b

#### TypeScript SDK Example - Ignore DeepInfra

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.3-70b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    ignore: ['deepinfra'],
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - Ignore DeepInfra

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'meta-llama/llama-3.3-70b-instruct',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      ignore: ['deepinfra'],
    },
  }),
});
```

#### Python Example - Ignore DeepInfra

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'meta-llama/llama-3.3-70b-instruct',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'ignore': ['deepinfra'],
  },
})
```

## Quantization

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `quantizations` | string[] | - | Quantization level filters |

### Quantization Levels

- `int4`: Integer (4 bit)
- `int8`: Integer (8 bit)
- `fp4`: Floating point (4 bit)
- `fp6`: Floating point (6 bit)
- `fp8`: Floating point (8 bit)
- `fp16`: Floating point (16 bit)
- `bf16`: Brain floating point (16 bit)
- `fp32`: Floating point (32 bit)
- `unknown`: Unknown

### Example: Requesting FP8 Quantization

#### TypeScript SDK Example - FP8 Quantization

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.1-8b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    quantizations: ['fp8'],
  },
  stream: false,
});
```

#### TypeScript (fetch) Example - FP8 Quantization

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'meta-llama/llama-3.1-8b-instruct',
    messages: [{ role: 'user', content: 'Hello' }],
    provider: {
      quantizations: ['fp8'],
    },
  }),
});
```

#### Python Example - FP8 Quantization

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'HTTP-Referer': '<YOUR_SITE_URL>',
  'X-Title': '<YOUR_SITE_NAME>',
  'Content-Type': 'application/json',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'meta-llama/llama-3.1-8b-instruct',
  'messages': [{ 'role': 'user', 'content': 'Hello' }],
  'provider': {
    'quantizations': ['fp8'],
  },
})
```

## Provider-Specific Headers

### Anthropic Beta Features

| Feature | Header Value | Description |
|---------|--------------|-------------|
| Fine-Grained Tool Streaming | `fine-grained-tool-streaming-2025-05-14` | Enables granular streaming events during tool calls |
| Interleaved Thinking | `interleaved-thinking-2025-05-14` | Allows thinking/reasoning interleaved with output |
| Structured Outputs | `structured-outputs-2025-11-13` | Enables strict tool use with schema validation |

#### Example: Enabling Fine-Grained Tool Streaming

##### TypeScript SDK Example - Fine-Grained Tool Streaming

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send(
  {
    model: 'anthropic/claude-sonnet-4.5',
    messages: [{ role: 'user', content: 'What is the weather in Tokyo?' }],
    tools: [
      {
        type: 'function',
        function: {
          name: 'get_weather',
          description: 'Get the current weather for a location',
          parameters: {
            type: 'object',
            properties: {
              location: { type: 'string' },
            },
            required: ['location'],
          },
        },
      },
    ],
    stream: true,
  },
  {
    headers: {
      'x-anthropic-beta': 'fine-grained-tool-streaming-2025-05-14',
    },
  },
);
```

##### TypeScript (fetch) Example - Fine-Grained Tool Streaming

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
    'x-anthropic-beta': 'fine-grained-tool-streaming-2025-05-14',
  },
  body: JSON.stringify({
    model: 'anthropic/claude-sonnet-4.5',
    messages: [{ role: 'user', content: 'What is the weather in Tokyo?' }],
    tools: [
      {
        type: 'function',
        function: {
          name: 'get_weather',
          description: 'Get the current weather for a location',
          parameters: {
            type: 'object',
            properties: {
              location: { type: 'string' },
            },
            required: ['location'],
          },
        },
      },
    ],
    stream: true,
  }),
});
```

##### Python Example - Fine-Grained Tool Streaming

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'Content-Type': 'application/json',
  'x-anthropic-beta': 'fine-grained-tool-streaming-2025-05-14',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'anthropic/claude-sonnet-4.5',
  'messages': [{ 'role': 'user', 'content': 'What is the weather in Tokyo?' }],
  'tools': [
    {
      'type': 'function',
      'function': {
        'name': 'get_weather',
        'description': 'Get the current weather for a location',
        'parameters': {
          'type': 'object',
          'properties': {
            'location': { 'type': 'string' },
          },
          'required': ['location'],
        },
      },
    },
  ],
  'stream': True,
})
```

#### Example: Enabling Interleaved Thinking

##### TypeScript SDK Example - Interleaved Thinking

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const completion = await openRouter.chat.send(
  {
    model: 'anthropic/claude-sonnet-4.5',
    messages: [{ role: 'user', content: 'Solve this step by step: What is 15% of 240?' }],
    stream: true,
  },
  {
    headers: {
      'x-anthropic-beta': 'interleaved-thinking-2025-05-14',
    },
  },
);
```

##### TypeScript (fetch) Example - Interleaved Thinking

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
    'x-anthropic-beta': 'interleaved-thinking-2025-05-14',
  },
  body: JSON.stringify({
    model: 'anthropic/claude-sonnet-4.5',
    messages: [{ role: 'user', content: 'Solve this step by step: What is 15% of 240?' }],
    stream: true,
  }),
});
```

##### Python Example - Interleaved Thinking

```python
import requests

headers = {
  'Authorization': 'Bearer <OPENROUTER_API_KEY>',
  'Content-Type': 'application/json',
  'x-anthropic-beta': 'interleaved-thinking-2025-05-14',
}

response = requests.post('https://openrouter.ai/api/v1/chat/completions', headers=headers, json={
  'model': 'anthropic/claude-sonnet-4.5',
  'messages': [{ 'role': 'user', 'content': 'Solve this step by step: What is 15% of 240?' }],
  'stream': True,
})
```

#### Combining Multiple Beta Features

```
x-anthropic-beta: fine-grained-tool-streaming-2025-05-14,interleaved-thinking-2025-05-14
```

## Key Considerations

- Automatic parameter support: Tool use and max_tokens requests automatically route only to compatible providers
- Load balancing disabled: When `sort` or `order` fields are set
- Fallback behavior: Soft constraints (performance thresholds) preserve reliability; hard constraints (pricing) may fail
- Account-wide settings: Privacy, provider filtering, and ZDR settings can be configured globally
