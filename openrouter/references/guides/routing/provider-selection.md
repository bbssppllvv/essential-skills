# Provider Routing Documentation

## Overview

OpenRouter intelligently routes requests to the best available providers for your model. By default, requests are load balanced across top providers to maximize uptime while prioritizing cost efficiency.

## Provider Configuration Object

The `provider` object in your request body allows customization of routing behavior with these fields:

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| `order` | string[] | - | Specify providers to try in order |
| `allow_fallbacks` | boolean | true | Allow backup providers when primary unavailable |
| `require_parameters` | boolean | false | Only use providers supporting all request parameters |
| `data_collection` | "allow" \| "deny" | "allow" | Control data retention policies |
| `zdr` | boolean | - | Restrict to Zero Data Retention endpoints |
| `enforce_distillable_text` | boolean | - | Restrict to distillation-allowed models |
| `only` | string[] | - | Allow specific providers exclusively |
| `ignore` | string[] | - | Skip specified providers |
| `quantizations` | string[] | - | Filter by quantization levels |
| `sort` | string \| object | - | Sort by price, throughput, or latency |
| `preferred_min_throughput` | number \| object | - | Minimum throughput threshold (tokens/sec) |
| `preferred_max_latency` | number \| object | - | Maximum latency threshold (seconds) |
| `max_price` | object | - | Maximum acceptable pricing |

## Default Load Balancing Strategy

OpenRouter's default approach prioritizes stability and cost:

1. Routes to providers without recent 30-second outages
2. Among stable providers, weights selection by inverse square of price
3. Uses remaining providers as fallbacks

**Example**: Provider A ($1/M tokens) is 9x more likely selected than Provider C ($3/M), because 1/(3^2) = 1/9.

## Provider Sorting

Disable load balancing by explicitly sorting providers:

- `"price"`: Lowest cost priority
- `"throughput"`: Highest speed priority
- `"latency"`: Lowest response time priority

**Shortcut modifiers**:
- `:nitro` appended to model slug = throughput sorting
- `:floor` appended to model slug = price sorting

## Advanced Sorting with Partitioning

When using model fallbacks, configure sorting as an object:

```json
{
  "sort": {
    "by": "throughput",
    "partition": "model"
  }
}
```

- `partition: "model"` (default) groups endpoints by model before sorting
- `partition: "none"` sorts globally across all models

This enables scenarios like routing to the highest-performing model regardless of which was listed first.

## Performance Thresholds

Set minimum throughput or maximum latency preferences with percentile-based metrics (p50, p75, p90, p99):

```json
{
  "preferred_min_throughput": { "p90": 50 },
  "preferred_max_latency": { "p90": 3 }
}
```

**Important**: These preferences are soft constraints -- providers missing thresholds become fallbacks rather than being excluded entirely, ensuring request reliability.

## Provider Selection Strategies

### Specific Provider Ordering

```typescript
provider: {
  order: ['anthropic', 'openai']
}
```

### Exclusive Provider Access

```typescript
provider: {
  only: ['azure']
}
```

### Ignoring Providers

```typescript
provider: {
  ignore: ['deepinfra']
}
```

### Cost-Optimized with Performance Floor

Combine price sorting with throughput thresholds to minimize costs while maintaining performance:

```typescript
provider: {
  sort: { by: 'price', partition: 'none' },
  preferred_min_throughput: { p90: 50 }
}
```

## Data & Privacy Controls

### Data Collection Policies

```typescript
provider: {
  data_collection: 'deny'  // Only providers not storing data
}
```

### Zero Data Retention (ZDR)

```typescript
provider: {
  zdr: true  // Only ZDR-compliant endpoints
}
```

### Distillation Permissions

```typescript
provider: {
  enforce_distillable_text: true  // Only distillation-allowed models
}
```

## Parameter Requirements

Enforce strict parameter support:

```typescript
provider: {
  require_parameters: true
}
response_format: { type: 'json_object' }
```

Only routes to providers supporting all specified request parameters.

## Quantization Filtering

Supported levels: `int4`, `int8`, `fp4`, `fp6`, `fp8`, `fp16`, `bf16`, `fp32`, `unknown`

```typescript
provider: {
  quantizations: ['fp8']
}
```

## Maximum Pricing

```typescript
provider: {
  max_price: {
    prompt: 1,      // $1 per million prompt tokens
    completion: 2   // $2 per million completion tokens
  }
}
```

This is a hard constraint -- requests fail if no provider meets pricing limits.

## Provider-Specific Features

### Anthropic Beta Headers

Pass beta features through the `x-anthropic-beta` header:

- `fine-grained-tool-streaming-2025-05-14`: Granular tool call streaming
- `interleaved-thinking-2025-05-14`: Interleaved reasoning output
- `structured-outputs-2025-11-13`: Strict tool use validation

```typescript
headers: {
  'x-anthropic-beta': 'fine-grained-tool-streaming-2025-05-14'
}
```

Multiple features: separate with commas.

## Key Considerations

- Automatic parameter support: Tool use and max_tokens requests automatically route only to compatible providers
- Load balancing disabled: When `sort` or `order` fields are set
- Fallback behavior: Soft constraints (performance thresholds) preserve reliability; hard constraints (pricing) may fail
- Account-wide settings: Privacy, provider filtering, and ZDR settings can be configured globally
