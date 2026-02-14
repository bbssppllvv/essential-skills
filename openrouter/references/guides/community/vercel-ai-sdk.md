# Vercel AI SDK Integration with OpenRouter

## Overview

The Vercel AI SDK enables developers to integrate OpenRouter with Next.js applications. This guide demonstrates the setup process and practical usage patterns.

## Installation

To get started, install the OpenRouter AI SDK provider package:

```bash
npm install @openrouter/ai-sdk-provider
```

## Basic Usage

The integration leverages Vercel's `streamText()` API to handle streaming responses from OpenRouter models.

## Code Examples

### Text Generation Example

```typescript
import { createOpenRouter } from '@openrouter/ai-sdk-provider';
import { streamText } from 'ai';

export const getLasagnaRecipe = async (modelName: string) => {
  const openrouter = createOpenRouter({
    apiKey: '${API_KEY_REF}',
  });

  const response = streamText({
    model: openrouter(modelName),
    prompt: 'Write a vegetarian lasagna recipe for 4 people.',
  });

  await response.consumeStream();
  return response.text;
};
```

### Tool Integration Example

```typescript
import { z } from 'zod';

export const getWeather = async (modelName: string) => {
  const openrouter = createOpenRouter({
    apiKey: '${API_KEY_REF}',
  });

  const response = streamText({
    model: openrouter(modelName),
    prompt: 'What is the weather in San Francisco, CA in Fahrenheit?',
    tools: {
      getCurrentWeather: {
        description: 'Get the current weather in a given location',
        parameters: z.object({
          location: z.string().describe('The city and state, e.g. San Francisco, CA'),
          unit: z.enum(['celsius', 'fahrenheit']).optional(),
        }),
        execute: async ({ location, unit = 'celsius' }) => {
          // Implementation with mock weather data
        },
      },
    },
  });

  await response.consumeStream();
  return response.text;
};
```
