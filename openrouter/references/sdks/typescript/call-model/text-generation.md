# Text Generation - OpenRouter SDK Documentation

## Overview
The OpenRouter SDK enables text generation through the `callModel` method, supporting various input formats, model configurations, and response patterns including text, streaming, and structured output.

## Basic Usage
The simplest implementation involves creating an OpenRouter instance and calling a model:

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openrouter = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Explain quantum computing in one sentence.',
});

const text = await result.getText();
```

## Input Format Options

**String Input**: Pass a simple string that becomes a user message.

**Message Array**: Enable multi-turn conversations by providing message objects with role and content fields.

**Multimodal Support**: Include images alongside text by using message arrays with input_text and input_image content types, allowing the model to analyze visual content.

## System Instructions
Configure model behavior using the `instructions` parameter to establish specific behavioral guidelines for the assistant.

## Model Selection

**Single Model**: Reference a model by its OpenRouter identifier.

**Model Fallback**: Provide an array of models; "The SDK will try each model in order until one succeeds."

## Response Handling

**getText()**: Retrieves only the text content after execution completes.

**getResponse()**: Returns the complete response object containing output array and usage metrics (input tokens, output tokens, and cached tokens).

## Generation Parameters
Control output characteristics through:
- Temperature (0-2 scale, affecting determinism versus creativity)
- maxOutputTokens (generation limit)
- topP (sampling parameter)

## Structured Output
Request JSON-formatted responses by specifying `format: { type: 'json_object' }` in the text configuration.

## Error Handling
Catch status codes (401 for invalid keys, 429 for rate limits, 503 for unavailability) to handle specific error scenarios appropriately.

## Concurrent Operations
Multiple independent `callModel` invocations can execute simultaneously using `Promise.all()` for parallel request handling.
