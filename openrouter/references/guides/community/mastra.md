---
title: Mastra
description: Integrate OpenRouter using Mastra framework for unified AI model access
---

# Mastra Integration

Integrate OpenRouter using Mastra framework for unified AI model access through a single interface.

## Setup

### Project Initialization

The quickest way to get started is using the Mastra CLI:

```bash
npx create-mastra@latest
```

This will guide you through prompts to set up your project.

### Environment Configuration

Add your credentials to `.env.development`:

```
# .env.development
# OPENAI_API_KEY=your-openai-key  # Comment out or remove this line
OPENROUTER_API_KEY=sk-or-your-api-key-here
```

### Install the Provider

If you previously had the OpenAI provider installed, remove it first:

```bash
npm uninstall @ai-sdk/openai
```

Then install the OpenRouter provider:

```bash
npm install @openrouter/ai-sdk-provider
```

## Agent Configuration

### Basic Agent Setup

Create your agent file at `src/mastra/agents/agent.ts`:

```typescript
import { Agent } from '@mastra/core/agent';
import { createOpenRouter } from '@openrouter/ai-sdk-provider';

// Initialize OpenRouter provider
const openrouter = createOpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

// Create an agent
export const assistant = new Agent({
  model: openrouter('anthropic/claude-3-opus'),
  name: 'Assistant',
  instructions:
    'You are a helpful assistant with expertise in technology and science.',
});
```

### Mastra Entry Point

Update your Mastra entry point at `src/mastra/index.ts`:

```typescript
import { Mastra } from '@mastra/core';

import { assistant } from './agents/agent'; // Update the import path if you used a different filename

export const mastra = new Mastra({
  agents: { assistant }, // Use the same name here as you exported from your agent file
});
```

## Running Your Application

Start the development server:

```bash
npm run dev
```

Your agent becomes accessible via:

- **REST endpoint**: `http://localhost:4111/api/agents/assistant/generate`
- **Interactive playground**: `http://localhost:4111`

### Testing with curl

```bash
curl -X POST http://localhost:4111/api/agents/assistant/generate \
-H "Content-Type: application/json" \
-d '{"messages": ["What are the latest advancements in quantum computing?"]}'
```

## Basic Integration Example

```typescript
import { Agent } from '@mastra/core/agent';
import { createOpenRouter } from '@openrouter/ai-sdk-provider';

// Initialize the OpenRouter provider
const openrouter = createOpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

// Create an agent using OpenRouter
const assistant = new Agent({
  model: openrouter('anthropic/claude-3-opus'),
  name: 'Assistant',
  instructions: 'You are a helpful assistant.',
});

// Generate a response
const response = await assistant.generate([
  {
    role: 'user',
    content: 'Tell me about renewable energy sources.',
  },
]);

console.log(response.text);
```

## Multi-Model Support

The framework allows creating multiple agents with different models, enabling model comparison and selection based on specific use cases:

```typescript
import { Agent } from '@mastra/core/agent';
import { createOpenRouter } from '@openrouter/ai-sdk-provider';

const openrouter = createOpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

// Create agents using different models
const claudeAgent = new Agent({
  model: openrouter('anthropic/claude-3-opus'),
  name: 'ClaudeAssistant',
  instructions: 'You are a helpful assistant powered by Claude.',
});

const gptAgent = new Agent({
  model: openrouter('openai/gpt-4'),
  name: 'GPTAssistant',
  instructions: 'You are a helpful assistant powered by GPT-4.',
});

// Use different agents based on your needs
const claudeResponse = await claudeAgent.generate([
  {
    role: 'user',
    content: 'Explain quantum mechanics simply.',
  },
]);
console.log(claudeResponse.text);

const gptResponse = await gptAgent.generate([
  {
    role: 'user',
    content: 'Explain quantum mechanics simply.',
  },
]);
console.log(gptResponse.text);
```

## Advanced Configuration

### Using extraBody for Provider-Specific Options

```typescript
import { Agent } from '@mastra/core/agent';
import { createOpenRouter } from '@openrouter/ai-sdk-provider';

// Initialize with advanced options
const openrouter = createOpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
  extraBody: {
    reasoning: {
      max_tokens: 10,
    },
  },
});

// Create an agent with model-specific options
const chefAgent = new Agent({
  model: openrouter('anthropic/claude-3.7-sonnet', {
    extraBody: {
      reasoning: {
        max_tokens: 10,
      },
    },
  }),
  name: 'Chef',
  instructions: 'You are a chef assistant specializing in French cuisine.',
});
```

### Using providerOptions for Caching and Other Features

```typescript
// Get a response with provider-specific options
const response = await chefAgent.generate([
  {
    role: 'system',
    content:
      'You are Chef Michel, a culinary expert specializing in ketogenic (keto) diet...',
    providerOptions: {
      // Provider-specific options - key can be 'anthropic' or 'openrouter'
      anthropic: {
        cacheControl: { type: 'ephemeral' },
      },
    },
  },
  {
    role: 'user',
    content: 'Can you suggest a keto breakfast?',
  },
]);
```

## Additional Resources

- [OpenRouter Documentation](https://openrouter.ai/docs)
- [Mastra Framework Documentation](https://mastra.ai/docs)
- [Vercel AI SDK Documentation](https://ai-sdk.dev)
