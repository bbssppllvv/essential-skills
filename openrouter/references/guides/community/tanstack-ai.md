# TanStack AI Integration with OpenRouter

## Overview

TanStack AI enables integration with OpenRouter, providing access to 300+ AI models from various providers through a unified API for React, Solid, and Preact applications.

## Installation

```bash
npm install @tanstack/ai @tanstack/ai-openrouter
```

## Basic Usage

The simplest implementation involves importing chat functionality and specifying an adapter:

```typescript
import { chat } from "@tanstack/ai";
import { openRouterText } from "@tanstack/ai-openrouter";

const stream = chat({
  adapter: openRouterText("openai/gpt-5.2"),
  messages: [{ role: "user", content: "Hello!" }],
});
```

## Configuration

Configure the adapter with API credentials and optional parameters:

```typescript
import { createOpenRouter, type OpenRouterConfig } from "@tanstack/ai-openrouter";

const config: OpenRouterConfig = {
  apiKey: process.env.OPENROUTER_API_KEY!,
  baseURL: "https://openrouter.ai/api/v1",
  httpReferer: "https://your-app.com",
  xTitle: "Your App Name",
};

const adapter = createOpenRouter(config.apiKey, config);
```

## Available Models

Models follow the format `provider/model-name`. Examples include:
- `"openai/gpt-5.2"`
- `"anthropic/claude-sonnet-4.5"`
- `"google/gemini-3-pro-preview"`
- `"z-ai/glm-4.7"`
- `"minimax/minimax-m2.1"`

## Server-Side Implementation

For server endpoints, convert streams to Server-Sent Events:

```typescript
import { chat, toServerSentEventsResponse } from "@tanstack/ai";
import { openRouterText } from "@tanstack/ai-openrouter";

export async function POST(request: Request) {
  const { messages } = await request.json();
  const stream = chat({
    adapter: openRouterText("openai/gpt-5.2"),
    messages,
  });
  return toServerSentEventsResponse(stream);
}
```

## Tool Usage

Define and implement tools with schema validation:

```typescript
import { chat, toolDefinition } from "@tanstack/ai";
import { openRouterText } from "@tanstack/ai-openrouter";
import { z } from "zod";

const getWeatherDef = toolDefinition({
  name: "get_weather",
  description: "Get the current weather",
  inputSchema: z.object({
    location: z.string(),
  }),
});

const stream = chat({
  adapter: openRouterText("openai/gpt-5.2"),
  messages: [{ role: "user", content: "What's the weather in NYC?" }],
  tools: [getWeatherDef.server(async ({ location }) => ({
    temperature: 72,
    conditions: "sunny",
  }))],
});
```

## Advanced Routing Features

### Model Fallbacks

```typescript
const stream = chat({
  adapter: openRouterText("openai/gpt-5.2"),
  messages: [{ role: "user", content: "Hello!" }],
  modelOptions: {
    models: ["anthropic/claude-sonnet-4.5", "google/gemini-3-pro-preview"],
    route: "fallback",
  },
});
```

### Provider Sorting

Sort by performance metrics rather than default load balancing:

```typescript
const stream = chat({
  adapter: openRouterText("openai/gpt-5.2"),
  messages: [{ role: "user", content: "Hello!" }],
  modelOptions: {
    provider: {
      sort: "throughput",
    },
  },
});
```

### Provider Ordering

Specify explicit provider sequence with fallback options:

```typescript
modelOptions: {
  provider: {
    order: ["anthropic", "amazon-bedrock", "google-vertex"],
    allow_fallbacks: true,
  },
}
```

### Data Privacy Controls

```typescript
modelOptions: {
  provider: {
    data_collection: "deny",
    zdr: true,
  },
}
```

### Provider Filtering

Include/exclude specific providers and quantization types:

```typescript
modelOptions: {
  provider: {
    only: ["together", "fireworks"],
    ignore: ["azure"],
    quantizations: ["fp16", "bf16"],
  },
}
```

### Cost Controls

Set maximum price limits per request:

```typescript
modelOptions: {
  provider: {
    max_price: {
      prompt: 0.5,
      completion: 2,
    },
  },
}
```

## Environment Setup

```bash
OPENROUTER_API_KEY=sk-or-...
```

## Resources

- TanStack AI Documentation
- OpenRouter Adapter Documentation
- OpenRouter Models Directory
