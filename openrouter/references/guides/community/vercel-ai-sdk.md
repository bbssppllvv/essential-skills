# Vercel AI SDK Integration with OpenRouter

## Overview

The Vercel AI SDK enables developers to integrate OpenRouter with Next.js applications using a dedicated provider package. Users can leverage OpenRouter's model access through the `@openrouter/ai-sdk-provider` package.

## Installation

Install the required package:

```bash
npm install @openrouter/ai-sdk-provider
```

## Implementation Examples

### Text Streaming Example

```typescript
import { createOpenRouter } from '@openrouter/ai-sdk-provider';
import { streamText } from 'ai';

export const getLasagnaRecipe = async (modelName: string) => {
  const openrouter = createOpenRouter({
    apiKey: '$OPENROUTER_API_KEY',
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
import { createOpenRouter } from '@openrouter/ai-sdk-provider';
import { streamText } from 'ai';
import { z } from 'zod';

export const getWeather = async (modelName: string) => {
  const openrouter = createOpenRouter({
    apiKey: '$OPENROUTER_API_KEY',
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
          const weatherData = {
            'Boston, MA': { celsius: '15°C', fahrenheit: '59°F' },
            'San Francisco, CA': { celsius: '18°C', fahrenheit: '64°F' },
          };
          const weather = weatherData[location];
          if (!weather) return `Weather data for ${location} is not available.`;
          return `The current weather in ${location} is ${weather[unit]}.`;
        },
      },
    },
  });

  await response.consumeStream();
  return response.text;
};
```

## Key Features

- Uses the `streamText()` API for streaming responses
- Supports tool/function calling with Zod schema validation
- Integrates seamlessly with Next.js applications
