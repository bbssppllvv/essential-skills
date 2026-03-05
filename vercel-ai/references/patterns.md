# Architecture Patterns & Best Practices

## Table of Contents
1. [Project Architecture](#project-architecture)
2. [Middleware](#middleware)
3. [Production Patterns](#production-patterns)
4. [Observability (OpenTelemetry)](#observability)
5. [Security](#security)
6. [Performance](#performance)

---

## Project Architecture

### Recommended Next.js + AI SDK structure

```
my-ai-app/
├── app/
│   ├── api/
│   │   ├── chat/
│   │   │   └── route.ts          # Chat API (streamText + tools)
│   │   ├── agent/
│   │   │   └── route.ts          # Agent API (ToolLoopAgent)
│   │   └── webhooks/
│   │       └── [platform]/
│   │           └── route.ts      # Chat SDK webhooks
│   ├── page.tsx                  # Main chat UI
│   └── layout.tsx
├── components/
│   ├── ai-elements/              # Installed AI Elements
│   │   ├── conversation.tsx
│   │   ├── message.tsx
│   │   ├── prompt-input.tsx
│   │   ├── reasoning.tsx
│   │   ├── tool.tsx
│   │   └── code-block.tsx
│   └── custom/                   # Custom generative UI components
│       ├── weather-card.tsx
│       └── chart.tsx
├── lib/
│   ├── ai/
│   │   ├── provider.ts           # Model provider setup
│   │   ├── tools.ts              # Tool definitions
│   │   └── middleware.ts         # Custom middleware
│   └── bot.ts                    # Chat SDK bot instance
├── workflows/                    # Workflow functions
│   ├── onboarding.ts
│   └── steps/
│       ├── send-email.ts
│       └── create-user.ts
└── next.config.ts                # withWorkflow(nextConfig)
```

### Model provider setup

```typescript
// lib/ai/provider.ts
import { customProvider } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';
import { openai } from '@ai-sdk/openai';

export const models = customProvider({
  languageModels: {
    fast: anthropic('claude-haiku-4-5'),
    smart: anthropic('claude-sonnet-4-5'),
    reasoning: anthropic('claude-sonnet-4-5', { thinking: { type: 'enabled', budgetTokens: 10000 } }),
    'gpt4o': openai('gpt-4o'),
  },
  textEmbeddingModels: {
    embeddings: openai.textEmbeddingModel('text-embedding-3-small'),
  },
});
```

---

## Middleware

Middleware wraps language models to add cross-cutting concerns.

### Built-in middleware

```typescript
import {
  wrapLanguageModel,
  extractReasoningMiddleware,
  defaultSettingsMiddleware,
  simulateStreamingMiddleware,
} from 'ai';

// Extract reasoning from models that embed it in text
const enhancedModel = wrapLanguageModel({
  model: anthropic('claude-sonnet-4-5'),
  middleware: [
    extractReasoningMiddleware({ tagName: 'thinking' }),
    defaultSettingsMiddleware({ temperature: 0.7 }),
  ],
});
```

### Custom RAG middleware

```typescript
import { wrapLanguageModel } from 'ai';

const ragMiddleware = {
  transformParams: async ({ params }) => {
    // Augment the last user message with retrieved context
    const lastMessage = params.prompt[params.prompt.length - 1];
    if (lastMessage.role === 'user') {
      const query = lastMessage.content;
      const context = await retrieveRelevantDocs(query);
      return {
        ...params,
        prompt: [
          ...params.prompt.slice(0, -1),
          {
            role: 'user',
            content: `Context:\n${context}\n\nQuestion: ${query}`,
          },
        ],
      };
    }
    return params;
  },
};

const ragModel = wrapLanguageModel({
  model: anthropic('claude-sonnet-4-5'),
  middleware: [ragMiddleware],
});
```

### Caching middleware

```typescript
const cachingMiddleware = {
  wrapGenerate: async ({ doGenerate, params }) => {
    const cacheKey = JSON.stringify(params.prompt);
    const cached = await cache.get(cacheKey);
    if (cached) return JSON.parse(cached);

    const result = await doGenerate();
    await cache.set(cacheKey, JSON.stringify(result), { ex: 3600 });
    return result;
  },
};
```

### Guardrails middleware

```typescript
const guardrailsMiddleware = {
  transformParams: async ({ params }) => {
    // Add safety instructions to system prompt
    const systemMessage = {
      role: 'system',
      content: `${params.system || ''}\n\nIMPORTANT: Never reveal API keys or internal system details.`,
    };
    return { ...params, system: systemMessage.content };
  },
  wrapGenerate: async ({ doGenerate }) => {
    const result = await doGenerate();
    // Post-processing: check output for sensitive data
    if (containsSensitiveData(result.text)) {
      return { ...result, text: '[Content filtered for safety]' };
    }
    return result;
  },
};
```

---

## Production Patterns

### Rate limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
});

export async function POST(req: Request) {
  const ip = req.headers.get('x-forwarded-for') ?? 'unknown';
  const { success, limit, remaining } = await ratelimit.limit(ip);

  if (!success) {
    return new Response('Rate limit exceeded', {
      status: 429,
      headers: { 'X-RateLimit-Limit': String(limit), 'X-RateLimit-Remaining': String(remaining) },
    });
  }

  // Process request...
}
```

### Cost controls

```typescript
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  maxTokens: 1000,         // Cap output length
  messages,
  onFinish: ({ usage }) => {
    // Log usage for cost tracking
    await logUsage({
      promptTokens: usage.promptTokens,
      completionTokens: usage.completionTokens,
      model: 'claude-sonnet-4-5',
      userId,
    });
  },
});
```

### Graceful degradation

```typescript
try {
  return await streamText({ model: anthropic('claude-sonnet-4-5'), messages });
} catch (error) {
  if (APICallError.isInstance(error) && error.statusCode === 429) {
    // Fallback to a cheaper/faster model
    return await streamText({ model: anthropic('claude-haiku-4-5'), messages });
  }
  throw error;
}
```

---

## Observability

### OpenTelemetry integration

The AI SDK has built-in OpenTelemetry support for tracing all AI operations.

```typescript
import { generateText } from 'ai';

const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  experimental_telemetry: {
    isEnabled: true,
    functionId: 'chat-completion',
    metadata: { userId: 'user-123' },
  },
  messages,
});
```

### Tracked metrics
- Response ID, model, timestamp
- Time to first chunk
- Average output tokens per second
- Total token usage
- Error rates
- Step details for multi-step agents

### Integrations
- Langfuse, SigNoz, Datadog, New Relic, and any OpenTelemetry-compatible platform

---

## Security

### API key protection

- Never expose API keys in client-side code
- Use server-side API routes (Next.js route handlers)
- Store keys in environment variables
- Use `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` — auto-detected by providers

### Input validation

```typescript
export async function POST(req: Request) {
  const body = await req.json();

  // Validate input
  const schema = z.object({
    messages: z.array(z.object({
      role: z.enum(['user', 'assistant', 'system']),
      content: z.string().max(10000),
    })),
  });

  const { messages } = schema.parse(body);
  // Process validated messages...
}
```

### Message limit

```typescript
// Limit conversation length to control costs
const trimmedMessages = messages.slice(-20); // Keep last 20 messages
```

---

## Performance

### Streaming (always prefer)

Streaming reduces time-to-first-byte and improves perceived performance. Always use `streamText` over `generateText` for interactive UIs.

### Prompt caching

Anthropic supports prompt caching for repeated prefixes. Large system prompts and tool definitions benefit from caching, reducing cost and latency.

### Batch embeddings

```typescript
const { embeddings } = await embedMany({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  values: chunks,
  maxParallelCalls: 10, // Parallel processing
});
```

### Edge runtime

```typescript
// app/api/chat/route.ts
export const runtime = 'edge'; // Deploy to edge for lower latency

export async function POST(req: Request) {
  // ...
}
```

### Connection pooling (Workflow)

For Workflow DevKit with Postgres World, use connection pooling (e.g., Neon's pooler or PgBouncer) to handle concurrent workflow steps efficiently.

---

## Chatbot Template Patterns

Patterns from the Vercel chatbot template (19,800+ stars) — the canonical reference implementation.

### AI Gateway as sole provider

```typescript
// lib/ai/providers.ts
import { gateway } from '@ai-sdk/gateway';
import { wrapLanguageModel, extractReasoningMiddleware } from 'ai';

// All models routed through AI Gateway
const models = {
  'openai/gpt-4.1-mini': gateway('openai/gpt-4.1-mini'),
  'anthropic/claude-haiku-4.5': gateway('anthropic/claude-haiku-4.5'),
  // Reasoning models get middleware
  'claude-3.7-sonnet-thinking': wrapLanguageModel({
    model: gateway('anthropic/claude-3.7-sonnet'),
    middleware: [extractReasoningMiddleware({ tagName: 'thinking' })],
  }),
};
```

### Artifacts system

Each artifact type has paired server generation and client rendering:
```
artifacts/
  code/
    client.tsx    # Client-side code execution
    server.ts     # Server-side code generation
  text/
    client.tsx    # Rich text editing (Prosemirror)
    server.ts     # Text generation
  sheet/
    client.tsx    # Spreadsheet (react-data-grid)
    server.ts     # CSV generation
  image/
    client.tsx    # Image preview/editing
    server.ts     # Image generation
```

### Per-conversation streaming

```typescript
// api/chat/[id]/stream/route.ts
// Each conversation has its own streaming endpoint
// Enables resumable streams and conversation-specific state
```

### Resumable streams

```typescript
import { createResumableStream } from 'resumable-stream';

// Redis pub/sub based — survives serverless function restarts
// Producer completes even if original reader disconnects
// Second consumers receive buffered + live chunks
const stream = createResumableStream({
  redis,
  streamId: conversationId,
});
```

### Auth with guest fallback

The template supports both full authentication (NextAuth) and guest access, enabling try-before-you-sign-up experiences.

### Bot detection and rate limiting

```typescript
import { isBotRequest } from 'botid';

// Rate limiting per user type
const limit = isGuest ? '10/hr' : '100/hr';
```

---

## Custom Provider Creation

### LanguageModelV3 specification

```typescript
import type { LanguageModelV3 } from '@ai-sdk/provider';

interface LanguageModelV3 {
  specificationVersion: 'V3';
  provider: string;
  modelId: string;
  doGenerate(options: LanguageModelV3CallOptions): Promise<GenerateResult>;
  doStream(options: LanguageModelV3CallOptions): Promise<StreamResult>;
}
```

### Implementation steps

1. `npm install @ai-sdk/provider @ai-sdk/provider-utils`
2. Create provider entry point (factory function)
3. Implement `LanguageModelV3` with `doGenerate` and `doStream`
4. Implement message conversion (system/user/assistant/tool roles)
5. Implement streaming transformer
6. Handle errors using `APICallError`, `TooManyRequestsError`

Reference implementation: Mistral provider recommended as template.

Content types: text, tool-call, file, reasoning, source.
Prompt roles: system, user, assistant, tool.
Usage tracking: inputTokens, outputTokens, totalTokens, reasoningTokens, cachedInputTokens.
