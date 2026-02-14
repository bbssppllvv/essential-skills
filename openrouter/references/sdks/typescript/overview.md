# OpenRouter TypeScript SDK Documentation

## Overview

The OpenRouter TypeScript SDK is a type-safe toolkit for building AI applications with access to 300+ language models through a unified API.

## Key Benefits

**Automatic API Updates**
The SDK is auto-generated from OpenRouter's OpenAPI specifications, ensuring new models and features become available immediately in IDE autocomplete without manual intervention.

**Type Safety**
Every parameter, response field, and configuration option is fully typed, preventing invalid configurations at compile time rather than runtime.

**Enhanced Error Handling**
The SDK provides contextual error messages. Rather than generic failures, developers receive specific guidance -- for example, explaining which models require minimum message counts and how many were provided.

**Type-Safe Streaming**
Streaming responses maintain full type information throughout consumption, allowing developers to access nested properties like `chunk.choices[0]?.delta?.content` with confidence.

## Installation & Setup

Installation requires a single npm command:
```bash
npm install @openrouter/sdk
```

API keys are obtained from openrouter.ai/settings/keys.

## Basic Usage Example

```typescript
import OpenRouter from '@openrouter/sdk';

const client = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY
});

const response = await client.chat.send({
  model: "minimax/minimax-m2",
  messages: [{ role: "user", content: "Hello!" }]
});

console.log(response.choices[0].message.content);
```

## Status

The SDK and documentation are currently in beta, with issue reporting available on GitHub.
