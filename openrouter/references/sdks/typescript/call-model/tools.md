# Tools Documentation - OpenRouter SDK

## Overview

The OpenRouter SDK enables developers to create type-safe tools using Zod schemas with automatic execution. It supports regular tools, generator tools with progress tracking, manual tools, and automatic multi-turn execution.

## The tool() Helper

The `tool()` function creates type-safe tools with Zod validation:

```typescript
import { OpenRouter, tool } from '@openrouter/sdk';
import { z } from 'zod';

const weatherTool = tool({
  name: 'get_weather',
  description: 'Get the current weather for a location',
  inputSchema: z.object({
    location: z.string().describe('City name, e.g., "San Francisco, CA"'),
  }),
  outputSchema: z.object({
    temperature: z.number(),
    conditions: z.string(),
  }),
  execute: async (params) => {
    const weather = await fetchWeather(params.location);
    return {
      temperature: weather.temp,
      conditions: weather.description,
    };
  },
});
```

## Tool Types

### Regular Tools

Standard tools with an execute function for basic operations.

### Generator Tools

Tools that yield progress updates during execution by including an `eventSchema`:

```typescript
const searchTool = tool({
  name: 'search_database',
  description: 'Search documents with progress updates',
  inputSchema: z.object({
    query: z.string(),
    limit: z.number().default(10),
  }),
  eventSchema: z.object({
    progress: z.number().min(0).max(100),
    message: z.string(),
  }),
  outputSchema: z.object({
    results: z.array(z.string()),
    totalFound: z.number(),
  }),
  execute: async function* (params) {
    yield { progress: 0, message: 'Starting search...' };
    // ... execution logic ...
    yield { progress: 100, message: 'Complete!' };
    return { results, totalFound };
  },
});
```

Progress events stream through `getToolStream()` and `getFullResponsesStream()`.

### Manual Tools

Tools requiring manual handling without automatic execution:

```typescript
const manualTool = tool({
  name: 'send_email',
  description: 'Send an email (requires user confirmation)',
  inputSchema: z.object({
    to: z.string().email(),
    subject: z.string(),
    body: z.string(),
  }),
  execute: false,
});
```

## Schema Definition

### Input Schema

Defines accepted parameters with optional defaults, enums, nested objects, and arrays.

### Output Schema

Specifies the structure returned to the model, including nested objects and arrays.

### Event Schema

For generator tools, defines progress/status event structures.

## Type Inference

Extract types from tool definitions:

```typescript
import type { InferToolInput, InferToolOutput, InferToolEvent } from '@openrouter/sdk';

type WeatherInput = InferToolInput<typeof weatherTool>;
type WeatherOutput = InferToolOutput<typeof weatherTool>;
type SearchEvent = InferToolEvent<typeof searchTool>;
```

## Using Tools with callModel

### Single Tool

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the weather in Tokyo?',
  tools: [weatherTool],
});

const text = await result.getText();
```

### Multiple Tools

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Search for TypeScript tutorials and calculate 2+2',
  tools: [searchTool, calculatorTool],
});
```

### Type-Safe Tool Calls

Use `as const` for full type inference:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the weather?',
  tools: [weatherTool, searchTool] as const,
  maxToolRounds: 0,
});

for await (const toolCall of result.getToolCallsStream()) {
  if (toolCall.name === 'get_weather') {
    console.log('Weather for:', toolCall.arguments.location);
  }
}
```

## TurnContext

Tool execute functions receive a `TurnContext` providing conversation state:

```typescript
const contextAwareTool = tool({
  name: 'context_tool',
  inputSchema: z.object({ data: z.string() }),
  outputSchema: z.object({ result: z.string() }),
  execute: async (params, context) => {
    console.log('Turn number:', context?.numberOfTurns);
    return { result: `Processed on turn ${context?.numberOfTurns}` };
  },
});
```

| Property | Type | Description |
|----------|------|-------------|
| `numberOfTurns` | number | Current turn number (1-indexed) |
| `turnRequest` | OpenResponsesRequest \| undefined | Current request object |
| `toolCall` | OpenResponsesFunctionToolCall \| undefined | Specific tool call being executed |

## Tool Execution

### Automatic Execution Flow

The SDK automatically executes tools and handles multi-turn conversations until the model provides a final response.

### Controlling Execution Rounds

#### maxToolRounds (Number)

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Research this topic thoroughly',
  tools: [searchTool, analyzeTool],
  maxToolRounds: 3,
});
```

Setting `maxToolRounds: 0` disables automatic execution.

#### maxToolRounds (Function)

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Research and analyze',
  tools: [searchTool],
  maxToolRounds: (context) => {
    return context.numberOfTurns < 5;
  },
});
```

## Accessing Tool Calls

### getToolCalls()

Retrieve all tool calls from the initial response:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the weather in Tokyo and Paris?',
  tools: [weatherTool],
  maxToolRounds: 0,
});

const toolCalls = await result.getToolCalls();

for (const call of toolCalls) {
  console.log(`Tool: ${call.name}`);
  console.log(`Arguments:`, call.arguments);
}
```

### getToolCallsStream()

Stream tool calls as they complete:

```typescript
for await (const toolCall of result.getToolCallsStream()) {
  console.log(`Received tool call: ${toolCall.name}`);
  const result = await processWeatherRequest(toolCall.arguments);
}
```

## Tool Stream Events

### getToolStream()

Stream argument deltas and preliminary results:

```typescript
for await (const event of result.getToolStream()) {
  switch (event.type) {
    case 'delta':
      process.stdout.write(event.content);
      break;
    case 'preliminary_result':
      console.log(`Progress: ${event.result.progress}%`);
      break;
  }
}
```

| Event Type | Description |
|------------|-------------|
| `delta` | Raw tool call argument chunks from model |
| `preliminary_result` | Progress events from generator tools |

### Tool Result Events

```typescript
for await (const event of result.getFullResponsesStream()) {
  if (event.type === 'tool.result') {
    console.log(`Tool ${event.toolCallId} completed`);
    console.log('Result:', event.result);
    if (event.preliminaryResults) {
      console.log('All progress events:', event.preliminaryResults);
    }
  }
}
```

#### ToolResultEvent Type

```typescript
type ToolResultEvent<TResult = unknown, TPreliminaryResults = unknown> = {
  type: 'tool.result';
  toolCallId: string;
  result: TResult;
  timestamp: number;
  preliminaryResults?: TPreliminaryResults[];
};
```

## Parallel Tool Execution

When the model calls multiple tools, they execute in parallel automatically:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Get weather in Paris, Tokyo, and New York simultaneously',
  tools: [weatherTool],
});

const text = await result.getText();
```

## Manual Tool Handling

For tools without execute functions:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Send an email to alice@example.com',
  tools: [confirmTool],
  maxToolRounds: 0,
});

const toolCalls = await result.getToolCalls();

for (const call of toolCalls) {
  if (call.name === 'send_email') {
    const confirmed = await showConfirmDialog(call.arguments);
    if (confirmed) {
      await sendEmail(call.arguments);
    }
  }
}
```

## Error Handling

### Tool Execution Errors

Errors in execute functions are caught and returned to the model:

```typescript
const riskyTool = tool({
  name: 'risky_operation',
  inputSchema: z.object({ input: z.string() }),
  outputSchema: z.object({ result: z.string() }),
  execute: async (params) => {
    if (params.input === 'fail') {
      throw new Error('Operation failed: invalid input');
    }
    return { result: 'success' };
  },
});
```

### Validation Errors

Invalid tool arguments are caught before execution based on Zod schema validation.

### Graceful Error Handling

```typescript
const robustTool = tool({
  name: 'fetch_data',
  inputSchema: z.object({ url: z.string().url() }),
  outputSchema: z.object({
    data: z.unknown().optional(),
    error: z.string().optional(),
  }),
  execute: async (params) => {
    try {
      const response = await fetch(params.url);
      if (!response.ok) {
        return { error: `HTTP ${response.status}: ${response.statusText}` };
      }
      return { data: await response.json() };
    } catch (error) {
      return { error: `Failed to fetch: ${error.message}` };
    }
  },
});
```

## Best Practices

### Descriptive Names and Descriptions

Use clear, specific naming conventions and detailed descriptions to help the model understand tool purpose and usage.

### Schema Descriptions

Add `.describe()` to parameters to help the model understand their purpose:

```typescript
const inputSchema = z.object({
  query: z.string().describe('Natural language search query'),
  maxResults: z.number()
    .min(1)
    .max(100)
    .default(10)
    .describe('Maximum number of results to return (1-100)'),
  dateRange: z.enum(['day', 'week', 'month', 'year', 'all'])
    .default('all')
    .describe('Filter results by time period'),
});
```

### Idempotent Tools

Design tools to be safely re-executable by checking for existing resources before creation.

### Timeout Handling

Wrap long-running operations with timeout mechanisms:

```typescript
const longRunningTool = tool({
  name: 'process_data',
  inputSchema: z.object({ dataId: z.string() }),
  execute: async (params) => {
    const timeoutMs = 30000;

    const result = await Promise.race([
      processData(params.dataId),
      new Promise((_, reject) =>
        setTimeout(() => reject(new Error('Operation timed out')), timeoutMs)
      ),
    ]);

    return result;
  },
});
```

## Next Steps

- **nextTurnParams** - Tool-driven context injection
- **Stop Conditions** - Advanced execution control
- **Examples** - Complete tool implementations
