# AI SDK UI Hooks Reference

## Table of Contents
1. [useChat](#usechat)
2. [Transport System](#transport-system)
3. [useCompletion](#usecompletion)
4. [useObject](#useobject)
5. [Message Types](#message-types)
6. [Streaming Patterns](#streaming-patterns)

---

## useChat

The primary hook for building chat interfaces. Manages message state, streaming, and API communication.

```tsx
'use client';
import { useChat } from '@ai-sdk/react';

export default function Chat() {
  const {
    messages,      // UIMessage[] — all messages
    sendMessage,   // (msg) => void — send user message (NOT append!)
    input,         // string — controlled input value
    setInput,      // (value) => void — update input
    status,        // 'ready' | 'submitted' | 'streaming' | 'error'
    isLoading,     // boolean — true while streaming
    error,         // Error | undefined
    stop,          // () => void — stop streaming
    reload,        // () => void — regenerate last response
    setMessages,   // (messages) => void — replace messages
    addToolApprovalResponse, // For tool approval workflows
  } = useChat({
    api: '/api/chat',           // API endpoint (default: '/api/chat')
    sendAutomaticallyWhen: ({ messages }) => {
      // Auto-send when last message has tool results (replaces removed maxSteps)
      const lastMsg = messages[messages.length - 1];
      return lastMsg?.parts.some(p => p.type.startsWith('tool-'));
    },
    onFinish: (message) => {},  // Called when response completes
    onError: (error) => {},     // Called on error
    onToolCall: async ({ toolCall }) => {
      // Handle client-side tool execution
      return result;
    },
  });

  return (
    <form onSubmit={(e) => { e.preventDefault(); sendMessage({ text: input }); }}>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button type="submit" disabled={status === 'streaming'}>Send</button>
    </form>
  );
}
```

### Sending messages

```tsx
// Simple text — v6 uses { text }, NOT { content }
sendMessage({ text: 'Hello' });

// With file attachments
sendMessage({
  text: 'What is in this image?',
  files: [
    { url: 'data:image/png;base64,...', mediaType: 'image/png' },
  ],
});

// With metadata
sendMessage({
  text: 'Hello',
  data: { customField: 'value' },
});
```

### Server-side API route

```typescript
// app/api/chat/route.ts
import { streamText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: anthropic('claude-sonnet-4-5'),
    system: 'You are a helpful assistant.',
    messages,
  });

  return result.toDataStreamResponse();
  // OR for UIMessage format:
  // return result.toUIMessageStreamResponse();
}
```

---

## Transport System

v6 introduces transport abstraction for controlling how useChat communicates with the backend.

### DefaultChatTransport (standard HTTP)

```tsx
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

const { messages, sendMessage } = useChat({
  transport: new DefaultChatTransport({ api: '/api/chat' }),
});
```

### DirectChatTransport (no HTTP)

Invoke agent directly without HTTP — useful for SSR, testing, or same-process execution:

```tsx
import { DirectChatTransport } from 'ai';

const { messages, sendMessage } = useChat({
  transport: new DirectChatTransport({
    handler: async ({ messages }) => {
      const result = await agent.stream({ messages });
      return result.toUIMessageStream();
    },
  }),
});
```

### TextStreamChatTransport

For simple text-only streaming:

```tsx
import { TextStreamChatTransport } from 'ai';

const { messages, sendMessage } = useChat({
  transport: new TextStreamChatTransport({ api: '/api/simple-chat' }),
});
```

### Custom transport

```typescript
const customTransport = {
  prepareSendMessagesRequest({ messages, body }) {
    // Customize request before sending
    return { url: '/api/chat', headers: { 'X-Custom': 'value' }, body: { ...body, messages } };
  },
};
```

---

## useCompletion

For single-turn text completion (not conversational).

```tsx
import { useCompletion } from '@ai-sdk/react';

const {
  completion,    // string — the generated text
  input,         // string — input value
  setInput,
  handleSubmit,  // form submit handler
  isLoading,
  error,
  stop,
} = useCompletion({
  api: '/api/completion',
});
```

---

## useObject

Streaming structured JSON objects with schema validation.

```tsx
'use client';
import { useObject } from '@ai-sdk/react';
import { z } from 'zod';

const schema = z.object({
  recipe: z.object({
    name: z.string(),
    ingredients: z.array(z.string()),
  }),
});

export default function RecipeGenerator() {
  const { object, submit, isLoading } = useObject({
    api: '/api/recipe',
    schema,
  });

  return (
    <div>
      <button onClick={() => submit('Generate a pasta recipe')}>
        Generate
      </button>
      {object && (
        <div>
          <h2>{object.recipe?.name}</h2>
          <ul>
            {object.recipe?.ingredients?.map((ing, i) => (
              <li key={i}>{ing}</li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );
}
```

Server route for useObject:

```typescript
// app/api/recipe/route.ts
import { streamText, Output } from 'ai';
import { z } from 'zod';

export async function POST(req: Request) {
  const { prompt } = await req.json();

  const result = streamText({
    model: anthropic('claude-sonnet-4-5'),
    prompt,
    output: Output.object({
      schema: z.object({
        recipe: z.object({
          name: z.string(),
          ingredients: z.array(z.string()),
        }),
      }),
    }),
  });

  return result.toDataStreamResponse();
}
```

---

## Message Types

### UIMessage (client-side)

The message type used by `useChat`. Messages contain a `parts` array for multi-modal content.

```typescript
interface UIMessage {
  id: string;
  role: 'user' | 'assistant' | 'system';
  parts: UIMessagePart[];  // Rich content parts (the only way to access content in v6)
  metadata?: Record<string, unknown>;
}

// v6 part types — note: tool parts use dynamic type names
type UIMessagePart =
  | { type: 'text'; text: string }
  | { type: 'reasoning'; text: string }                    // NOTE: property is `text`, NOT `reasoning`
  | { type: `tool-${string}`; toolCallId: string; args: any; state: ToolState; output?: any }
  | { type: 'source-url'; url: string; title?: string }    // NOT generic 'source'
  | { type: 'source-document'; document: any }
  | { type: 'file'; data: string; mimeType: string }
  | { type: 'step-start' }                                 // Step boundary marker
  | { type: `data-${string}`; data: any };                 // Custom data parts

type ToolState = 'input-streaming' | 'input-available' | 'approval-requested' | 'approval-responded' | 'output-available' | 'output-denied' | 'output-error';
```

> **Important v6 change**: UIMessage no longer has a `content` property. Always use `message.parts` for rendering. Tool part types use the pattern `tool-{toolName}` (e.g., `tool-getWeather`) rather than a generic `tool-invocation` type. Check with `part.type.startsWith('tool-')` to detect any tool part.

### Rendering message parts

Always iterate over `message.parts` and switch on `part.type`:

```tsx
{message.parts.map((part, i) => {
  // v6: tool parts use dynamic types like 'tool-getWeather'
  if (part.type === 'text') {
    return <MessageResponse key={i}>{part.text}</MessageResponse>;
  }
  if (part.type === 'reasoning') {
    return <Reasoning key={i}><ReasoningContent>{part.text}</ReasoningContent></Reasoning>;
  }
  if (part.type.startsWith('tool-')) {
    if (part.type === 'tool-getWeather' && part.state === 'output-available') {
      return <WeatherCard key={i} data={part.output} />;
    }
    return <Tool key={i} part={part} />;
  }
  if (part.type === 'source-url') {
    return <SourceLink key={i} url={part.url} title={part.title} />;
  }
  return null;
})}
```

### ModelMessage (server-side)

```typescript
// Convert between UI and Model messages
import { convertToModelMessages } from 'ai';

const modelMessages = await convertToModelMessages(uiMessages);
```

> **v6 Breaking Change**: `convertToModelMessages()` is now **async** — you must `await` it:
> ```typescript
> // v6 — MUST await
> const modelMessages = await convertToModelMessages(uiMessages);
> // v5 (old) — was synchronous
> // const modelMessages = convertToModelMessages(uiMessages);
> ```
> Also renamed: `CoreMessage` → `ModelMessage`, `convertToCoreMessages` → `convertToModelMessages`.

### Type Inference

```typescript
import type { InferUITool, InferUITools } from 'ai';

// Infer tool types from agent/tool definitions
type MyToolPart = InferUITool<typeof myAgent>;
type AllToolParts = InferUITools<typeof myAgent>;

// Generic useChat for typed messages
const { messages } = useChat<MyCustomMessage>();
```

---

## Streaming Patterns

### Data Stream Protocol

The AI SDK uses Server-Sent Events (SSE) with typed JSON chunks for client-server communication. The protocol supports 20+ part types including text deltas, tool calls, reasoning, sources, files, and custom data.

> For the complete protocol specification including all part types, SSE format, and guidance on building custom/mobile clients, see `references/data-stream-protocol.md`.

### Response methods

| Method | Use case |
|--------|----------|
| `toDataStreamResponse()` | Standard streaming for useChat |
| `toUIMessageStreamResponse()` | UIMessage-aware streaming |
| `toTextStreamResponse()` | Plain text streaming |
| `createAgentUIStreamResponse()` | For ToolLoopAgent |

### Custom streaming data

```typescript
import { streamText, createDataStream } from 'ai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const dataStream = createDataStream({
    execute: async (writer) => {
      // Send custom data to client
      writer.writeData({ customKey: 'value' });

      const result = streamText({
        model: anthropic('claude-sonnet-4-5'),
        messages,
      });

      result.mergeIntoDataStream(dataStream);
    },
  });

  return dataStream.toDataStreamResponse();
}
```

### Consuming custom data on the client

```tsx
const { messages, data } = useChat({
  onData: (data) => {
    // Handle custom data from server
    console.log('Custom data:', data);
  },
});
```
