# AI Gateway Reference

## Table of Contents
1. [Overview](#overview)
2. [Setup](#setup)
3. [Model Routing](#model-routing)
4. [Provider Fallbacks](#provider-fallbacks)
5. [Caching](#caching)
6. [BYOK (Bring Your Own Key)](#byok-bring-your-own-key)
7. [Observability & Cost Tracking](#observability--cost-tracking)
8. [Compatible APIs](#compatible-apis)
9. [AI SDK Integration](#ai-sdk-integration)

---

## Overview

Vercel AI Gateway is a unified API proxy providing access to hundreds of AI models from 40+ providers through a single endpoint. Instead of managing individual provider SDKs and API keys, you route all requests through one gateway.

- **Single endpoint**: `https://ai-gateway.vercel.sh`
- **Single API key**: `AI_GATEWAY_API_KEY` for all providers
- **Model identifier format**: `provider/model-name` (e.g. `openai/gpt-5.2`, `anthropic/claude-sonnet-4.5`)
- **40+ providers**: OpenAI, Anthropic, Google, DeepSeek, Groq, xAI, Mistral, Cohere, Amazon Bedrock, Azure, and many more
- **Pricing**: No markup on tokens. List price from providers. $5 monthly credits for new users.

---

## Setup

### Installation

```bash
pnpm add ai @ai-sdk/gateway
```

### Environment variables

Set `AI_GATEWAY_API_KEY` in `.env.local`:

```env
AI_GATEWAY_API_KEY=your-key-here
```

On Vercel, authentication is automatic via OIDC -- no manual key configuration needed.

### Basic usage

The simplest way to call a model through the gateway uses the `provider/model-name` string directly:

```typescript
import { streamText } from 'ai';

const result = streamText({
  model: 'openai/gpt-5.2',
  prompt: 'Hello',
});
```

Or use the gateway provider explicitly for more control:

```typescript
import { gateway } from '@ai-sdk/gateway';

const result = streamText({
  model: gateway('anthropic/claude-sonnet-4.5'),
  prompt: 'Hello',
});
```

---

## Model Routing

### Provider ordering

Control which underlying provider serves the request. The gateway tries providers in the specified order and falls back automatically on failure.

```typescript
const result = streamText({
  model: 'anthropic/claude-sonnet-4.5',
  prompt,
  providerOptions: {
    gateway: {
      order: ['bedrock', 'anthropic'],  // Try Bedrock first, fallback to Anthropic
    },
  },
});
```

### Provider restrictions

Restrict requests to a specific set of providers:

```typescript
const result = streamText({
  model: 'anthropic/claude-sonnet-4.5',
  prompt,
  providerOptions: {
    gateway: {
      only: ['bedrock', 'anthropic'],  // Only allow these two providers
    },
  },
});
```

Both `order` and `only` can be combined:

```typescript
providerOptions: {
  gateway: {
    order: ['bedrock', 'anthropic'],
    only: ['bedrock', 'anthropic'],
  },
}
```

---

## Provider Fallbacks

### Model-level fallbacks

Define a list of fallback models. If the primary model fails, the gateway tries each fallback in order:

```typescript
const result = streamText({
  model: 'openai/gpt-5.2',
  prompt,
  providerOptions: {
    gateway: {
      models: ['anthropic/claude-sonnet-4.5', 'google/gemini-3-flash'],
    },
  },
});
```

The gateway attempts `openai/gpt-5.2` first, then `anthropic/claude-sonnet-4.5`, then `google/gemini-3-flash`.

---

## Caching

Enable automatic provider-specific caching (e.g. Anthropic prompt caching) without manual configuration:

```typescript
const result = streamText({
  model: 'anthropic/claude-sonnet-4.5',
  prompt,
  providerOptions: {
    gateway: {
      caching: 'auto',
    },
  },
});
```

The `'auto'` setting handles provider-specific caching details automatically.

---

## BYOK (Bring Your Own Key)

Use your own provider API keys through the gateway. This lets you keep gateway benefits (routing, fallbacks, observability) while using your own billing accounts:

```typescript
const result = streamText({
  model: 'anthropic/claude-sonnet-4.5',
  prompt,
  providerOptions: {
    gateway: {
      byok: {
        anthropic: [{ apiKey: process.env.ANTHROPIC_API_KEY }],
      },
    },
  },
});
```

Multiple keys per provider are supported as an array.

---

## Observability & Cost Tracking

The gateway provides built-in observability features:

- **Usage graphs**: Requests by model, time-to-first-token (TTFT), token counts, spend
- **Request logs**: Full request/response logging
- **API key tracking**: Per-key usage monitoring
- **Generation IDs**: Unique identifiers for each generation

### Cost metadata

Access cost information from provider metadata after a request:

```typescript
const result = await generateText({
  model: 'openai/gpt-5.2',
  prompt: 'Hello',
});

// Access cost data from provider metadata
const cost = result.providerMetadata?.gateway?.cost;
const marketCost = result.providerMetadata?.gateway?.marketCost;
```

- `gateway.cost` -- Actual cost incurred
- `gateway.marketCost` -- List price cost from the provider

---

## Compatible APIs

The gateway exposes multiple API-compatible endpoints:

| Protocol | Endpoint |
|---|---|
| OpenAI-compatible | `https://ai-gateway.vercel.sh/v1` |
| Anthropic-compatible | `https://ai-gateway.vercel.sh` |
| OpenResponses | `https://ai-gateway.vercel.sh/v1/responses` |

This means you can use the gateway with:

- **AI SDK** (recommended, shown above)
- **OpenAI SDK** -- point the base URL to the gateway
- **Anthropic SDK** -- point the base URL to the gateway
- **Raw HTTP / cURL** -- call the endpoints directly

---

## AI SDK Integration

The `@ai-sdk/gateway` package provides first-class integration with the AI SDK. All standard AI SDK functions work seamlessly:

```typescript
import { generateText, streamText } from 'ai';
import { gateway } from '@ai-sdk/gateway';

// generateText
const { text } = await generateText({
  model: gateway('openai/gpt-5.2'),
  prompt: 'Explain quantum computing.',
});

// streamText
const stream = streamText({
  model: gateway('anthropic/claude-sonnet-4.5'),
  prompt: 'Write a story.',
});

// With full options
const result = streamText({
  model: gateway('anthropic/claude-sonnet-4.5'),
  prompt: 'Complex task',
  providerOptions: {
    gateway: {
      order: ['bedrock', 'anthropic'],
      models: ['google/gemini-3-flash'],
      caching: 'auto',
      byok: {
        anthropic: [{ apiKey: process.env.ANTHROPIC_API_KEY }],
      },
    },
  },
});
```

All `providerOptions.gateway` fields can be combined as needed.
