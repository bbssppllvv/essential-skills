# Call Model (TypeScript) Documentation

## Overview

The OpenRouter SDK's `callModel` function provides a unified API for calling any LLM with automatic tool execution and multiple consumption patterns. This feature enables developers to access over 300 language models through a single interface.

## Key Features

- **Items-Based Architecture**: Built on OpenRouter's Responses API with structured items (messages, tool calls, reasoning) rather than raw message chunks
- **Flexible Response Handling**: Developers can retrieve text, stream responses, or access structured data from a single call
- **Automatic Tool Execution**: The SDK manages tool execution loops when tools are defined using Zod schemas
- **TypeScript Support**: Full type inference is provided for tool inputs, outputs, and events
- **Format Conversion**: Compatibility with OpenAI chat and Anthropic Claude message formats
- **Stream-First Design**: Supports concurrent consumers through a reusable streaming architecture

## Basic Usage

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openrouter = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the capital of France?',
});

const text = await result.getText();
console.log(text);
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

## Documentation Structure

The complete guide includes sections on items-based streaming, text generation, streaming patterns, tool creation, message format conversion, dynamic parameters, stop conditions management, and ready-to-use tool examples.
