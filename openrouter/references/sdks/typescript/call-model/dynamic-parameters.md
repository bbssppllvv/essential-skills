# Dynamic Parameters | OpenRouter SDK

## Overview

Dynamic parameters enable adaptive model behavior by using async functions to compute `callModel` parameter values based on conversation context. This allows developers to modify model selection, temperature, instructions, and other settings as conversations evolve.

## Core Concept

Any parameter in `callModel` accepts a function instead of a static value:

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openrouter = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

const result = openrouter.callModel({
  model: (ctx) => {
    return ctx.numberOfTurns > 3 ? 'openai/gpt-5.2' : 'openai/gpt-5-nano';
  },
  input: 'Hello!',
  tools: [myTool],
});
```

## Function Signature

Parameter functions receive a `TurnContext` and can return synchronous or asynchronous values:

```typescript
type ParameterFunction<T> = (context: TurnContext) => T | Promise<T>;
```

### TurnContext Properties

| Property | Type | Purpose |
|----------|------|---------|
| `numberOfTurns` | number | Current turn number (1-indexed) |
| `turnRequest` | OpenResponsesRequest \| undefined | Current request with messages and settings |
| `toolCall` | OpenResponsesFunctionToolCall \| undefined | Specific tool being executed |

## Async Functions for External Data

Functions can fetch data asynchronously:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  temperature: async (ctx) => {
    const prefs = await fetchUserPreferences(userId);
    return prefs.preferredTemperature ?? 0.7;
  },
  instructions: async (ctx) => {
    const rules = await fetchBusinessRules();
    return `Follow these rules:\n${rules.join('\n')}`;
  },
  input: 'Hello!',
});
```

## Implementation Patterns

### Progressive Model Upgrade

Transition from efficient to capable models based on conversation depth:

```typescript
const result = openrouter.callModel({
  model: (ctx) => {
    if (ctx.numberOfTurns <= 2) {
      return 'openai/gpt-5-nano';
    }
    return 'openai/gpt-5.2';
  },
  input: 'Let me think through this problem...',
  tools: [analysisTool],
});
```

### Adaptive Temperature

Adjust response creativity based on task indicators in messages:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  temperature: (ctx) => {
    const lastMessage = JSON.stringify(ctx.turnRequest?.input).toLowerCase();
    if (lastMessage.includes('creative') || lastMessage.includes('brainstorm')) {
      return 1.0;
    }
    if (lastMessage.includes('code') || lastMessage.includes('calculate')) {
      return 0.2;
    }
    return 0.7;
  },
  input: 'Write a creative story',
});
```

### Context-Aware Instructions

Customize system instructions based on conversation metrics:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  instructions: (ctx) => {
    const base = 'You are a helpful assistant.';
    const turnInfo = `This is turn ${ctx.numberOfTurns} of the conversation.`;
    if (ctx.numberOfTurns > 5) {
      return `${base}\n${turnInfo}\nKeep responses concise - this is a long conversation.`;
    }
    return `${base}\n${turnInfo}`;
  },
  input: 'Continue helping me...',
  tools: [helpTool],
});
```

### Dynamic Max Tokens

Vary output length based on request keywords:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  maxOutputTokens: (ctx) => {
    const lastMessage = JSON.stringify(ctx.turnRequest?.input).toLowerCase();
    if (lastMessage.includes('summarize') || lastMessage.includes('brief')) {
      return 200;
    }
    if (lastMessage.includes('detailed') || lastMessage.includes('explain')) {
      return 2000;
    }
    return 500;
  },
  input: 'Give me a detailed explanation',
});
```

### Feature Flags

Conditionally enable provider-specific features:

```typescript
const result = openrouter.callModel({
  model: 'anthropic/claude-sonnet-4.5',
  provider: async (ctx) => {
    const enableThinking = ctx.numberOfTurns > 2;
    return enableThinking ? {
      anthropic: {
        thinking: { type: 'enabled', budgetTokens: 1000 },
      },
    } : undefined;
  },
  input: 'Solve this complex problem',
  tools: [analysisTool],
});
```

## Integration with Tools

Dynamic parameters work with tool execution:

```typescript
const smartAssistant = openrouter.callModel({
  model: (ctx) => {
    const hasToolUse = JSON.stringify(ctx.turnRequest?.input).includes('function_call');
    return hasToolUse ? 'anthropic/claude-sonnet-4.5' : 'openai/gpt-5-nano';
  },
  temperature: (ctx) => {
    return ctx.numberOfTurns > 1 ? 0.3 : 0.7;
  },
  input: 'Research and analyze this topic',
  tools: [searchTool, analysisTool],
});
```

## Execution Sequence

Parameters resolve at the beginning of each turn:

1. Resolve all parameter functions using current TurnContext
2. Build request with resolved values
3. Send to model
4. Execute tools if needed
5. Check stop conditions
6. Update TurnContext for next turn
7. Repeat from step 1

## Error Handling

Implement fallbacks for failed async operations:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  instructions: async (ctx) => {
    try {
      const rules = await fetchRules();
      return `Follow these rules: ${rules}`;
    } catch (error) {
      console.error('Failed to fetch rules:', error);
      return 'You are a helpful assistant.';
    }
  },
  input: 'Hello!',
});
```

## Best Practices

### Keep Functions Pure

Avoid side effects within parameter functions:

```typescript
// Recommended: Pure function
model: (ctx) => ctx.numberOfTurns > 3 ? 'gpt-4' : 'gpt-4o-mini',

// Not recommended: Contains side effect
model: (ctx) => {
  logToDatabase(ctx);
  return 'gpt-4';
},
```

### Cache Expensive Operations

Store computed values to avoid repeated fetching:

```typescript
let cachedRules: string | null = null;

const result = openrouter.callModel({
  instructions: async (ctx) => {
    if (!cachedRules) {
      cachedRules = await fetchExpensiveRules();
    }
    return cachedRules;
  },
  input: 'Hello!',
});
```

### Provide Sensible Defaults

Always include fallback values for missing data:

```typescript
model: (ctx) => {
  const preferredModel = getPreferredModel();
  return preferredModel ?? 'openai/gpt-5-nano';
},
```

## Related Resources

- **nextTurnParams** - Tool-driven parameter modification
- **Stop Conditions** - Dynamic execution control
- **Tools** - Multi-turn orchestration
