# Streaming Documentation - OpenRouter SDK

## Overview
The OpenRouter SDK provides multiple patterns for consuming streaming responses in real-time, built on a reusable stream architecture supporting concurrent consumers.

## Text Streaming

**getTextStream()** delivers text content as it's generated, yielding small chunks (typically a few characters or words):

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Write a short poem about the ocean.',
});

for await (const delta of result.getTextStream()) {
  process.stdout.write(delta);
}
```

## Reasoning Streaming

**getReasoningStream()** streams the reasoning process for models supporting extended thinking (o1, Claude with thinking):

```typescript
const result = openrouter.callModel({
  model: 'openai/o1-preview',
  input: 'Solve this step by step: If x + 5 = 12, what is x?',
});

console.log('Reasoning:');
for await (const delta of result.getReasoningStream()) {
  process.stdout.write(delta);
}
```

## Items Streaming (Recommended)

**getItemsStream()** provides the recommended approach for structured access to all output types. Each iteration yields a complete item with updated content—replace items by ID rather than accumulating deltas.

Supported item types:
- `message` - Assistant text responses
- `function_call` - Tool invocations with arguments
- `reasoning` - Model thinking
- `web_search_call` - Web search operations
- `file_search_call` - File search operations
- `image_generation_call` - Image generation operations
- `function_call_output` - Tool execution results

```typescript
for await (const item of result.getItemsStream()) {
  switch (item.type) {
    case 'message':
      console.log('Message:', item.content);
      break;
    case 'function_call':
      console.log('Tool call:', item.name, item.arguments);
      break;
  }
}
```

## Message Streaming (Deprecated)

**getNewMessagesStream()** is deprecated; use `getItemsStream()` instead for all item types.

## Full Event Streaming

**getFullResponsesStream()** streams all response events including tool preliminary results:

Key event types:
- `response.created` - Response object created
- `response.output_text.delta` - Text content chunk
- `response.reasoning.delta` - Reasoning content chunk
- `response.function_call_arguments.delta` - Tool call argument chunk
- `response.completed` - Full response complete
- `tool.preliminary_result` - Progress from generator tools
- `tool.result` - Final result from tool execution

## Tool Call Streaming

**getToolCallsStream()** delivers structured tool calls as they complete:

```typescript
for await (const toolCall of result.getToolCallsStream()) {
  console.log(`Tool: ${toolCall.name}`);
  console.log(`Arguments:`, toolCall.arguments);
}
```

**getToolStream()** streams tool deltas and preliminary results from generator tools.

## Concurrent Consumers

Multiple consumers can read from the same result simultaneously using the reusable stream architecture, ensuring each consumer receives all events.

## Cancellation

Cancel streaming to stop generation:

```typescript
if (charCount > 500) {
  await result.cancel();
  break;
}
```

## UI Framework Integration

Implementations available for React (with state management and cleanup) and Server-Sent Events (SSE) via Hono framework for server-side streaming.
