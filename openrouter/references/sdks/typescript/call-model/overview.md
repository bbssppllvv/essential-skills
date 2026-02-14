# Call Model (TypeScript) Documentation

## Overview

The OpenRouter SDK's `callModel` function offers a unified interface for accessing 300+ language models with built-in support for tool execution and multiple response consumption patterns.

## Key Features

- **Items-Based Architecture**: Built on OpenRouter's Responses API with structured items (messages, tool calls, reasoning)
- **Flexible Response Handling**: Developers can retrieve text, stream responses, or access structured data from a single call
- **Automatic Tool Execution**: The SDK manages tool execution loops when tools are defined using Zod schemas
- **TypeScript Support**: Full type inference is provided for tool inputs, outputs, and events
- **Format Conversion**: Compatibility with OpenAI chat and Anthropic Claude message formats
- **Stream-First Design**: Supports concurrent consumers through a reusable streaming architecture

## Basic Usage

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the capital of France?',
});

const text = await result.getText();
```

## Response Consumption Methods

**Text retrieval:**
```typescript
const text = await result.getText();
const response = await result.getResponse(); // includes usage data
```

**Streaming options:**
- Text delta streaming via `getTextStream()`
- Reasoning streams for reasoning models
- Item-based streaming with `getItemsStream()`
- Complete event streams with `getFullResponsesStream()`

**Tool operations:**
- Retrieve all tool calls with `getToolCalls()`
- Stream tool calls as they complete
- Monitor deltas and preliminary results

## Input Flexibility

The function accepts strings, message arrays, or configurations with system instructions for customized interactions.

## Additional Resources

The documentation points to comprehensive guides covering items-based streaming, text generation, streaming patterns, tool creation, message format conversion, dynamic parameters, execution controls, and ready-to-use tool examples.
