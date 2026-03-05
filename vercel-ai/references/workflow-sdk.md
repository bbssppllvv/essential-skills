# Workflow DevKit Reference

Build durable, resumable TypeScript workflows with `"use workflow"` and `"use step"` directives.
**Package**: `npm install workflow@latest`
**Docs**: https://useworkflow.dev/

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Setup](#setup)
3. [Steps](#steps)
4. [Sleep & Waiting](#sleep--waiting)
5. [Webhooks & Hooks](#webhooks--hooks)
6. [Error Handling](#error-handling)
7. [AI Integration (DurableAgent)](#ai-integration)
8. [Deployment & Worlds](#deployment--worlds)
9. [Observability](#observability)

---

## Core Concepts

### Directives

- **`"use workflow"`** — Marks a function as durable. It can pause, resume, and survive restarts.
- **`"use step"`** — Marks a retryable unit of work. Runs once, result is persisted, can be retried on failure.

Both are compiled by an SWC plugin at build time into an execution graph where each step becomes an isolated API route.

### How it works

1. Workflow function is the orchestrator
2. Each step compiles to a separate API route
3. Inputs/outputs are recorded (persisted)
4. On crash/deploy, system replays deterministically
5. While a step runs, the workflow is suspended (zero resource consumption)

---

## Setup

### Next.js

```bash
npm install workflow@latest
```

```typescript
// next.config.ts
import { withWorkflow } from 'workflow/next';
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {};
export default withWorkflow(nextConfig);
```

### Other frameworks
Supported: Next.js, Vite, Nuxt, Express, Fastify, Hono, Nitro, SvelteKit, Astro, NestJS (experimental). Coming soon: TanStack Start, React Router.

---

## Steps

```typescript
import { sleep } from 'workflow';

export async function userOnboarding(email: string) {
  'use workflow';

  const user = await createUser(email);       // Step
  await sendWelcomeEmail(user);               // Step
  await sleep('3 days');                       // Pauses workflow, zero resources
  await sendFollowUpEmail(user);              // Step
  return { userId: user.id, status: 'done' };
}

// Step function
async function createUser(email: string) {
  'use step';
  const user = await db.users.create({ email });
  return user;
}

async function sendWelcomeEmail(user: User) {
  'use step';
  await emailService.send({ to: user.email, subject: 'Welcome!' });
}
```

### maxRetries

```typescript
// Default: maxRetries = 3 (4 total attempts: 1 initial + 3 retries)
createUser.maxRetries = 5;  // 6 total attempts
sendEmail.maxRetries = 0;   // Run once, no retry
```

---

## Sleep & Waiting

```typescript
import { sleep } from 'workflow';

export async function dripCampaign(email: string) {
  'use workflow';

  await sendEmail(email, 'Welcome!');
  await sleep('5s');        // 5 seconds
  await sleep('3 days');    // 3 days
  await sleep('1 week');    // 1 week
  await sendEmail(email, 'We miss you!');
}
```

Sleep pauses the workflow without consuming compute resources. State is persisted. Accepts duration strings (`'5s'`, `'3 days'`), milliseconds, or `Date` objects.

> **Important**: `sleep()` must be called from within a workflow context (`"use workflow"`), NOT from within a step (`"use step"`). This is a common mistake.

---

## Webhooks & Hooks

### Webhooks (external event waiting)

```typescript
import { createWebhook } from 'workflow';

export async function paymentFlow(orderId: string) {
  'use workflow';

  const order = await validateOrder(orderId);
  const webhook = createWebhook();
  await sendPaymentRequest(order, webhook.url); // Give URL to external service
  const confirmation = await webhook;            // Pause until webhook is called
  await fulfillOrder(order, confirmation);
}
```

### Hooks (typed resume with validation)

```typescript
import { defineHook } from 'workflow';

const approvalHook = defineHook({
  schema: z.object({ approved: z.boolean(), reason: z.string().optional() }),
});

export async function approvalWorkflow(requestId: string) {
  'use workflow';

  const request = await loadRequest(requestId);
  await notifyApprover(request, approvalHook.token);
  const decision = await approvalHook; // Pause until triggered
  if (decision.approved) {
    await processApproval(request);
  } else {
    await processRejection(request, decision.reason);
  }
}
```

### Hooks with create/resume pattern

```typescript
const approvalHook = defineHook<{
  decision: 'approved' | 'changes';
  notes?: string;
}>();

export async function approvalWorkflow(topic: string) {
  'use workflow';
  const draft = await generateDraft(topic);
  const events = approvalHook.create({ token: 'draft-123' });
  for await (const event of events) {
    if (event.decision === 'approved') {
      await publishDraft(draft);
      break;
    }
  }
}

// Resume externally:
await approvalHook.resume('draft-123', { decision: 'approved' });
```

---

## Error Handling

### Error types

```typescript
import { FatalError, RetryableError } from 'workflow';

async function callAPI(data: any) {
  'use step';

  const resp = await fetch('https://api.example.com', {
    method: 'POST',
    body: JSON.stringify(data),
  });

  if (resp.status === 401) {
    throw new FatalError('Auth failed — do not retry');
  }

  if (resp.status === 429) {
    throw new RetryableError('Rate limited', {
      retryAfter: '5m',  // Duration string, ms, or Date
    });
  }

  return resp.json();
}
```

| Error | Behavior |
|-------|----------|
| `FatalError` | Stops immediately, no retry |
| `RetryableError` | Retries with exponential backoff until maxRetries |
| Unhandled error | Treated as retryable, uses default retry behavior |

### Custom retry delay

```typescript
throw new RetryableError('Temporarily unavailable', {
  retryAfter: (metadata.attempt ** 2) * 1000, // Exponential backoff in ms
});
```

---

## AI Integration

### DurableAgent

```typescript
import { DurableAgent } from '@workflow/ai';
import { anthropic } from '@ai-sdk/anthropic';
import { tool } from 'ai';

const agent = new DurableAgent({
  model: anthropic('claude-sonnet-4-5'),
  system: 'You are a research assistant.', // DurableAgent uses `system`, unlike ToolLoopAgent which uses `instructions`
  tools: {
    searchWeb: tool({ /* ... */ }),
    readPage: tool({ /* ... */ }),
  },
});

export async function researchWorkflow(question: string) {
  'use workflow';

  const result = await agent.stream({
    messages: [{ role: 'user', content: question }],
  });

  return result.messages;
}
```

**Important**: Override `globalThis.fetch` with the workflow's durable fetch before using AI SDK functions:

```typescript
export async function aiWorkflow(prompt: string) {
  'use workflow';
  // The durable fetch is automatically available inside "use workflow"
  const result = await generateText({
    model: anthropic('claude-sonnet-4-5'),
    prompt,
  });
  return result.text;
}
```

### DurableAgent Advanced

**Stream reconnection**: DurableAgent supports automatic stream reconnection for handling Vercel Function timeouts:

```typescript
import { WorkflowChatTransport } from '@workflow/ai';

// Client-side: auto-reconnecting transport
const { messages, sendMessage } = useChat({
  transport: new WorkflowChatTransport({
    api: '/api/workflow-chat',
    // Automatically reconnects using x-workflow-run-id headers
    // and startIndex query params for chunk resumption
  }),
});
```

**prepareStep callback**: Dynamic per-step agent configuration:

```typescript
const agent = new DurableAgent({
  model: anthropic('claude-sonnet-4-5'),
  system: 'Research assistant',
  tools: { searchWeb, readPage },
  prepareStep: async ({ stepIndex, messages }) => {
    // Dynamically adjust tools or system prompt per step
    return {
      tools: stepIndex > 5 ? { summarize } : { searchWeb, readPage },
    };
  },
});
```

**Multi-provider support**: DurableAgent supports both AI SDK v5 (LanguageModelV2) and v6 (LanguageModelV3) through a compatibility layer.

**collectUIMessages option**: Accumulate `UIMessage[]` during streaming for direct use with useChat.

---

## Deployment & Worlds

### Worlds architecture

| World | Use case |
|-------|----------|
| **Local World** | Development — virtual infrastructure, no provisioning |
| **Vercel World** | Production on Vercel — zero config, automatic |
| **Postgres World** | Self-hosted — Drizzle ORM + graphile-worker for job queues |
| **Community Worlds** | Custom infrastructure (Jazz, Turso, MongoDB, Redis, etc.) |

### Deploying to Vercel

1. Deploy normally — workflows auto-detect Vercel World
2. Enable Fluid compute
3. No extra configuration needed

### Self-hosted with Postgres

```bash
npm install @workflow/world-postgres
```

Uses Drizzle ORM for database operations and pg-boss for job queue management.

---

## Observability

### CLI

```bash
npx workflow inspect runs                    # List recent runs
npx workflow inspect runs --backend vercel   # Inspect Vercel runs
npx workflow inspect runs --web              # Launch web UI
npx workflow inspect --help                  # All commands
```

### What's tracked
- Workflow runs (real-time)
- Individual steps with inputs/outputs
- Webhooks and hooks
- Event log
- Stream output
- Failures and error traces
- Performance metrics

### Vercel Dashboard
Workflow observability is built into the Vercel dashboard on your project page.

---

## Use Cases

| Pattern | Description |
|---------|-------------|
| **User onboarding** | Welcome email → wait 3 days → follow-up → wait 7 days → push notification |
| **Email drip campaigns** | Multi-step campaigns with delays, A/B testing |
| **Payment processing** | Wait for payment webhook, confirm, fulfill |
| **AI agents** | Long-running agents that survive crashes |
| **RAG pipelines** | Multi-hour data ingestion with crash survival |
| **Integration flows** | Coordinate APIs with automatic retries |
