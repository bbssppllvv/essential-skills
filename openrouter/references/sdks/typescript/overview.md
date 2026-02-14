# OpenRouter TypeScript SDK Documentation

## Overview

The OpenRouter TypeScript SDK provides a type-safe toolkit for building AI applications with access to 300+ language models through a unified API.

## Key Features

**Auto-generated from API specifications:** The SDK is auto-generated from OpenRouter's OpenAPI specifications, ensuring new models and features become available immediately in IDE autocomplete without manual version management.

**Type-safe by default:** Every parameter, response field, and configuration option is fully typed, preventing invalid configurations at compile time rather than runtime. The SDK provides contextual error messages with specific guidance rather than generic failures -- for example, explaining which models require minimum message counts and how many were provided.

**Type-safe streaming:** Streaming responses maintain full type information throughout the iteration process, allowing developers to access nested properties like `chunk.choices[0]?.delta?.content` with confidence.

## Installation

```bash
npm install @openrouter/sdk
```

Obtain an API key from [openrouter.ai/settings/keys](https://openrouter.ai/settings/keys).

## Basic Usage

```typescript
import OpenRouter from '@openrouter/sdk';

const client = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY
});

const response = await client.chat.send({
  model: "minimax/minimax-m2",
  messages: [
    { role: "user", content: "Hello!" }
  ]
});

console.log(response.choices[0].message.content);
```

## Streaming Example

```typescript
const stream = await client.chat.send({
  model: "minimax/minimax-m2",
  messages: [{ role: "user", content: "Write a story" }],
  stream: true
});

for await (const chunk of stream) {
  const content = chunk.choices[0]?.delta?.content;
}
```

## Status

The TypeScript SDK and documentation are currently in beta. Report issues on [GitHub](https://github.com/OpenRouterTeam/typescript-sdk/issues).
