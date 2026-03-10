---
name: vercel-ai
description: "Complete Vercel AI ecosystem guide: AI SDK v6 (generateText, streamText, useChat, tools, agents, MCP), AI Gateway (routing, fallbacks, caching), AI Elements (48+ React UI components), Chat SDK (Slack/Discord/Teams bots), Workflow DevKit, Data Stream Protocol, and Security (Vercel Firewall/WAF, BotID bot detection, rate limiting, DDoS, Attack Challenge Mode, prompt injection prevention, cost controls). Use when building AI chat UIs, agents, streaming, tool calling, or working with ai/@ai-sdk packages. Also use when securing AI apps: Firewall rules, BotID, @vercel/firewall rate limiting, bot blocking, geo-blocking, IP blocking, AI endpoint protection, Spend Management, or any Vercel security for AI products."
---

# Vercel AI Ecosystem \u2014 Complete Implementation Guide

This skill covers the full Vercel AI stack for building production-grade AI applications:

| Layer | Package | Purpose |
|-------|---------|---------|
| **AI SDK Core** | `ai` | Text generation, streaming, structured output, tools, agents, embeddings |
| **AI SDK UI** | `@ai-sdk/react` | React hooks: `useChat`, `useCompletion`, `useObject` |
| **AI Gateway** | `@ai-sdk/gateway` | Unified model access, routing, fallbacks, caching, observability |
| **AI Elements** | `ai-elements` | 48+ pre-built React components for AI interfaces (shadcn/ui based) |
| **Chat SDK** | `chat` | Cross-platform bots (Slack, Discord, Teams, GitHub, Telegram, Linear) |
| **Workflow DevKit** | `workflow` | Durable, resumable workflows with `"use workflow"` / `"use step"` directives |
| **Security** | `botid`, `@vercel/firewall` | Firewall/WAF, BotID bot detection, rate limiting, DDoS, abuse protection |

## How to Use This Skill

1. Start with the Quick Start Patterns below for common use cases
2. Read the Critical v6 API Rules to avoid the most common mistakes
3. Consult reference files (below) for detailed API docs on specific domains

## Reference Files

Read reference files as needed for the specific domain you're working in:

| File | When to read |
|------|-------------|
| `references/ai-sdk-core.md` | generateText, streamText, Output helpers, generateImage, generateSpeech, providers, embeddings, error handling |
| `references/ai-sdk-ui.md` | useChat, useCompletion, useObject hooks, streaming patterns, message types |
| `references/ai-elements.md` | UI components: Message, Conversation, PromptInput, Reasoning, Tool, CodeBlock, etc. |
| `references/agents-and-tools.md` | ToolLoopAgent, multi-step agents, tool calling, MCP integration |
| `references/chat-sdk.md` | Cross-platform chat bots: adapters, cards, modals, streaming to platforms |
| `references/workflow-sdk.md` | Durable workflows, steps, sleep, webhooks, hooks, error handling, deployment |
| `references/patterns.md` | Architecture decisions, best practices, production patterns, middleware |
| `references/ai-gateway.md` | AI Gateway: routing, fallbacks, caching, BYOK, observability |
| `references/data-stream-protocol.md` | SSE protocol for custom backends and native mobile clients (SwiftUI) |
| `references/streamdown.md` | Streamdown markdown renderer: full API, plugins, remend, performance |
| `references/security.md` | Firewall/WAF rules, BotID, rate limiting, DDoS, prompt injection, cost protection |

## Quick Start Patterns

### Pattern 1: AI Chat with Beautiful UI (Most Common)

Server route + useChat hook + AI Elements components:

```typescript
// app/api/chat/route.ts
import { streamText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({
    model: anthropic('claude-sonnet-4-5'),
    messages,
  });
  return result.toDataStreamResponse();
}
```

```tsx
// app/page.tsx
'use client';
import { useChat } from '@ai-sdk/react';
import { Conversation, ConversationContent } from '@/components/ai-elements/conversation';
import { Message, MessageContent, MessageResponse } from '@/components/ai-elements/message';
import { PromptInput, PromptInputTextarea, PromptInputSubmit } from '@/components/ai-elements/prompt-input';

export default function Chat() {
  const { messages, sendMessage, status, input, setInput } = useChat();
  return (
    <div className="flex flex-col h-screen">
      <Conversation>
        <ConversationContent>
          {messages.map((message) => (
            <Message from={message.role} key={message.id}>
              <MessageContent>
                {message.parts.map((part, i) => {
                  if (part.type === 'text') {
                    return <MessageResponse key={`${message.id}-${i}`}>{part.text}</MessageResponse>;
                  }
                  return null;
                })}
              </MessageContent>
            </Message>
          ))}
        </ConversationContent>
      </Conversation>
      <PromptInput onSubmit={(value) => sendMessage({ text: value })}>
        <PromptInputTextarea value={input} onChange={(e) => setInput(e.target.value)} />
        <PromptInputSubmit />
      </PromptInput>
    </div>
  );
}
```

### Pattern 2: Agent with Tool Display + Reasoning

```tsx
// Render multi-part messages with reasoning, tools, and text
// v6: use if/else with startsWith for tool parts (dynamic type names)
{message.parts.map((part, i) => {
  if (part.type === 'text') {
    return <MessageResponse key={i}>{part.text}</MessageResponse>;
  }
  if (part.type === 'reasoning') {
    return (
      <Reasoning key={i}>
        <ReasoningTrigger />
        <ReasoningContent>{part.text}</ReasoningContent>
      </Reasoning>
    );
  }
  if (part.type.startsWith('tool-')) {
    return <Tool key={i} part={part} />;
  }
  return null;
})}
```

### Pattern 3: Structured Output with Tools

```typescript
import { streamText, tool, Output, stepCountIs } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';
import { z } from 'zod';

const result = await streamText({
  model: anthropic('claude-sonnet-4-5'),
  tools: {
    getWeather: tool({
      description: 'Get weather for a city',
      inputSchema: z.object({ city: z.string() }),
      execute: async ({ city }) => ({ temp: 72, condition: 'sunny' }),
    }),
  },
  output: Output.object({
    schema: z.object({
      summary: z.string(),
      recommendations: z.array(z.string()),
    }),
  }),
  stopWhen: stepCountIs(5), // Enable multi-step tool calling
  messages,
});
```

### Pattern 4: Cross-Platform Chat Bot

```typescript
import { Chat } from 'chat';
import { createSlackAdapter } from '@chat-adapter/slack';
import { createRedisState } from '@chat-adapter/state-redis';
import { streamText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const bot = new Chat({
  userName: 'mybot',
  adapters: { slack: createSlackAdapter() },
  state: createRedisState(),
});

bot.onNewMention(async (thread) => {
  await thread.subscribe();
  const result = streamText({
    model: anthropic('claude-sonnet-4-5'),
    prompt: thread.message.text,
  });
  await thread.post(result.toTextStream());
});
```

### Pattern 5: Durable Workflow

```typescript
import { sleep } from 'workflow';

export async function userOnboarding(email: string) {
  'use workflow';
  const user = await createUser(email);
  await sendWelcomeEmail(user);
  await sleep('3 days');
  await sendFollowUpEmail(user);
}
```

### Pattern 6: Secure AI Chat Endpoint (BotID + Rate Limiting + Auth)

```typescript
// instrumentation-client.ts — client-side BotID setup
import { initBotId } from 'botid/client/core';
initBotId({
  protect: [{ path: '/api/chat', method: 'POST' }],
});
```

```typescript
// app/api/chat/route.ts — layered server-side protection
import { checkBotId } from 'botid/server';
import { checkRateLimit } from '@vercel/firewall';
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';
import { auth } from '@/auth';

export const maxDuration = 30;

export async function POST(req: Request) {
  // Layer 1: Authentication
  const session = await auth();
  if (!session) return new Response('Unauthorized', { status: 401 });

  // Layer 2: Bot detection
  const { isBot } = await checkBotId();
  if (isBot) return Response.json({ error: 'Access denied' }, { status: 403 });

  // Layer 3: Rate limiting (per user)
  const { rateLimited } = await checkRateLimit('ai-chat-limit', {
    request: req,
    rateLimitKey: session.user.id,
  });
  if (rateLimited) return Response.json({ error: 'Rate limit exceeded' }, { status: 429 });

  // Layer 4: Input sanitization + resource limits
  const { messages } = await req.json();
  const sanitized = messages.filter((m: { role: string }) => m.role !== 'system');

  const result = streamText({
    model: openai('gpt-4o'),
    system: 'You are a helpful assistant.',
    messages: sanitized,
    maxTokens: 4096,
    maxSteps: 5,
    abortSignal: req.signal,
  });
  return result.toDataStreamResponse();
}
```

## Critical v6 API Rules

v6 renamed and restructured many APIs from v5. These are the most common mistakes \u2014 using old names causes runtime errors:

1. **Tool definition**: Use `tool()` helper with `inputSchema` (NOT `parameters` \u2014 renamed in v5/v6)
2. **Model specification**: Use provider functions \u2014 `anthropic('claude-sonnet-4-5')`, `openai('gpt-4o')`
3. **Chat hook**: Use `sendMessage({ text: '...' })` (NOT `append()`, NOT `{ content }` \u2014 v6 uses `text` property)
4. **Message format**: Access `message.parts` array, switch on `part.type`. UIMessage no longer has `content`.
5. **Reasoning parts**: Use `part.text` (NOT `part.reasoning` \u2014 property renamed to `text` in v6)
6. **Multi-step control**: Use `stopWhen: stepCountIs(N)` (NOT `maxSteps` \u2014 removed in v6). For useChat, use `sendAutomaticallyWhen`.
7. **Streaming response**: Use `toDataStreamResponse()` or `toUIMessageStreamResponse()`
8. **Agent response**: Standalone `createAgentUIStreamResponse({ agent, uiMessages })` (NOT a method on the agent)
9. **ToolLoopAgent**: Use `instructions` parameter (NOT `system` \u2014 renamed for agents)
10. **Structured output**: `Output.object()`, `Output.array()`, `Output.choice()`, `Output.json()`, `Output.text()`
11. **Tool approval**: Use `needsApproval: true` on tool definition (NOT `experimental_toolCallApproval`)
12. **Tool part types**: Parts use `tool-{toolName}` pattern. States: `input-streaming`, `input-available`, `approval-requested`, `output-available`
13. **Embeddings**: Use `provider.textEmbeddingModel('model-name')`
14. **MCP**: Use `createMCPClient()` from `@ai-sdk/mcp`
15. **Package manager**: Always detect from lockfile (pnpm-lock.yaml \u2192 pnpm, etc.)
16. **Message types**: `CoreMessage` renamed to `ModelMessage`. Use `convertToModelMessages()` (NOT `convertToCoreMessages` \u2014 renamed and now **async**)
17. **generateObject/streamObject**: Deprecated \u2014 use `generateText`/`streamText` with `Output.*` helpers instead
18. **Embeddings**: `textEmbeddingModel()` renamed to `embeddingModel()`, `textEmbedding()` to `embedding()`
19. **Token usage**: `cachedInputTokens` \u2192 `inputTokenDetails.cacheReadTokens`, `reasoningTokens` \u2192 `outputTokenDetails.reasoningTokens`
20. **OpenAI strict mode**: `strictJsonSchema` defaults to `true` in v6 \u2014 use `.nullable()` not `.optional()` in Zod schemas
21. **Azure**: `azure()` uses Responses API by default; use `azure.chat()` for Chat Completions. Metadata key: `azure` (not `openai`)
22. **Migration tool**: Run `npx @ai-sdk/codemod v6` to automate most v5\u21926 changes

## Installing AI Elements

```bash
# Individual components (recommended)
npx ai-elements@latest add message conversation prompt-input reasoning tool code-block

# All components via shadcn registry
npx shadcn@latest add https://elements.ai-sdk.dev/api/registry/all.json
```

Components install as source code into your project (typically `@/components/ai-elements/`), giving you full control. Requires Tailwind CSS in CSS Variables mode.

## Key Streaming Architecture

```
Client (useChat) <--SSE--> Server (streamText) <--Gateway--> LLM Provider
       |                        |                    |
  AI Elements              Middleware           AI Gateway
  (rendering)         (caching, logging,    (routing, fallbacks,
                       guardrails, RAG)      caching, observability)
```

The Data Stream Protocol uses Server-Sent Events with typed chunks: text deltas, tool calls, tool results, reasoning, and finish signals. MessageResponse component from AI Elements handles incremental markdown rendering via Streamdown without re-parsing on each chunk.
