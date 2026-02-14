# Streaming Documentation - OpenRouter SDK

## Overview

The OpenRouter SDK provides multiple streaming patterns for consuming LLM responses in real-time. All streams use a reusable architecture supporting concurrent consumers.

## Text Streaming

**getTextStream()** delivers text content as it's generated, yielding small chunks (typically a few characters or words):

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openrouter = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

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

console.log('\n\nFinal answer:');
const text = await result.getText();
console.log(text);
```

## Items Streaming (Recommended)

**getItemsStream()** provides the recommended approach for structured access to all output types. Each iteration yields a complete item with updated content -- replace items by ID rather than accumulating deltas.

Supported item types:
- `message` - Assistant text responses
- `function_call` - Tool invocations with arguments
- `reasoning` - Model thinking
- `web_search_call` - Web search operations
- `file_search_call` - File search operations
- `image_generation_call` - Image generation operations
- `function_call_output` - Tool execution results

```typescript
import type { StreamableOutputItem } from '@openrouter/sdk';

const result = openrouter.callModel({
  model: 'anthropic/claude-sonnet-4',
  input: 'Hello!',
  tools: [myTool],
});

for await (const item of result.getItemsStream()) {
  switch (item.type) {
    case 'message':
      console.log('Message:', item.content);
      break;
    case 'function_call':
      console.log('Tool call:', item.name, item.arguments);
      break;
    case 'reasoning':
      console.log('Thinking:', item.summary);
      break;
    case 'function_call_output':
      console.log('Tool result:', item.output);
      break;
  }
}
```

## Message Streaming (Deprecated)

**getNewMessagesStream()** is deprecated; use `getItemsStream()` instead for all item types.

## Full Event Streaming

**getFullResponsesStream()** streams all response events including tool preliminary results:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Search for documents',
  tools: [searchTool],
});

for await (const event of result.getFullResponsesStream()) {
  switch (event.type) {
    case 'response.output_text.delta':
      process.stdout.write(event.delta);
      break;
    case 'response.function_call_arguments.delta':
      console.log('Tool argument delta:', event.delta);
      break;
    case 'response.completed':
      console.log('Response complete');
      break;
    case 'tool.preliminary_result':
      console.log('Progress:', event.result);
      break;
    case 'tool.result':
      console.log('Tool completed:', event.toolCallId);
      console.log('Result:', event.result);
      if (event.preliminaryResults) {
        console.log('Preliminary results:', event.preliminaryResults);
      }
      break;
  }
}
```

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
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the weather in Paris and Tokyo?',
  tools: [weatherTool],
  maxToolRounds: 0,
});

for await (const toolCall of result.getToolCallsStream()) {
  console.log(`Tool: ${toolCall.name}`);
  console.log(`Arguments:`, toolCall.arguments);
  console.log(`ID: ${toolCall.id}`);
}
```

**getToolStream()** streams tool deltas and preliminary results from generator tools:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Search for TypeScript tutorials',
  tools: [searchTool],
});

for await (const event of result.getToolStream()) {
  if (event.type === 'delta') {
    process.stdout.write(event.content);
  } else if (event.type === 'preliminary_result') {
    console.log(`\nProgress (${event.toolCallId}):`, event.result);
  }
}
```

## Concurrent Consumers

Multiple consumers can read from the same result simultaneously using the reusable stream architecture, ensuring each consumer receives all events:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Write a story.',
});

const [text, response] = await Promise.all([
  (async () => {
    let text = '';
    for await (const delta of result.getTextStream()) {
      text += delta;
    }
    return text;
  })(),
  result.getResponse(),
]);

console.log('Text length:', text.length);
console.log('Token usage:', response.usage);
```

## Cancellation

Cancel streaming to stop generation:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Write a very long essay...',
});

const streamPromise = (async () => {
  let charCount = 0;
  for await (const delta of result.getTextStream()) {
    process.stdout.write(delta);
    charCount += delta.length;

    if (charCount > 500) {
      await result.cancel();
      break;
    }
  }
})();

await streamPromise;
console.log('\nCancelled!');
```

## UI Framework Integration

### React Example

```typescript
import { useState, useEffect } from 'react';

function ChatResponse({ prompt }: { prompt: string }) {
  const [text, setText] = useState('');
  const [isStreaming, setIsStreaming] = useState(true);

  useEffect(() => {
    const openrouter = new OpenRouter({ apiKey: process.env.REACT_APP_API_KEY });

    const result = openrouter.callModel({
      model: 'openai/gpt-5-nano',
      input: prompt,
    });

    (async () => {
      for await (const delta of result.getTextStream()) {
        setText(prev => prev + delta);
      }
      setIsStreaming(false);
    })();

    return () => {
      result.cancel();
    };
  }, [prompt]);

  return (
    <div>
      <p>{text}</p>
      {isStreaming && <span className="cursor">|</span>}
    </div>
  );
}
```

### Server-Sent Events (SSE) via Hono

```typescript
import { Hono } from 'hono';
import { streamSSE } from 'hono/streaming';

const app = new Hono();

app.get('/stream', (c) => {
  return streamSSE(c, async (stream) => {
    const result = openrouter.callModel({
      model: 'openai/gpt-5-nano',
      input: c.req.query('prompt') || 'Hello!',
    });

    for await (const delta of result.getTextStream()) {
      await stream.writeSSE({
        data: JSON.stringify({ delta }),
        event: 'delta',
      });
    }

    await stream.writeSSE({
      data: JSON.stringify({ done: true }),
      event: 'done',
    });
  });
});
```
