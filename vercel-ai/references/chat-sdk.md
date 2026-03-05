# Chat SDK Reference

Unified TypeScript SDK for building cross-platform chat bots.
**Package**: `npm install chat`
**Docs**: https://chat-sdk.dev/

## Table of Contents
1. [Architecture](#architecture)
2. [Setup](#setup)
3. [Event Handlers](#event-handlers)
4. [Cards & Modals](#cards--modals)
5. [AI SDK Integration](#ai-sdk-integration)
6. [Platform Adapters](#platform-adapters)
7. [Deployment](#deployment)

---

## Architecture

Write bot logic once, deploy to Slack, Teams, Discord, GitHub, Telegram, Google Chat, and Linear.

```
Chat Instance
  ├── Adapters (per-platform webhook handling)
  │   ├── Slack (createSlackAdapter)
  │   ├── Discord (createDiscordAdapter)
  │   ├── Teams (createTeamsAdapter)
  │   ├── Google Chat (createGChatAdapter)
  │   ├── Telegram (createTelegramAdapter)
  │   ├── GitHub (createGitHubAdapter)
  │   └── Linear (createLinearAdapter)
  ├── State Adapter (Redis for serverless persistence)
  └── Event Handlers (onNewMention, onSubscribedMessage, etc.)
```

---

## Setup

```bash
npm install chat @chat-adapter/slack @chat-adapter/state-redis
```

```typescript
// lib/bot.ts
import { Chat } from 'chat';
import { createSlackAdapter } from '@chat-adapter/slack';
import { createRedisState } from '@chat-adapter/state-redis';

export const bot = new Chat({
  userName: 'mybot',
  adapters: {
    slack: createSlackAdapter(),  // Auto-detects SLACK_BOT_TOKEN, SLACK_SIGNING_SECRET
  },
  state: createRedisState(),     // Auto-detects REDIS_URL
});
```

### Environment variables (auto-detected)

| Platform | Variables |
|----------|-----------|
| Slack | `SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET` |
| Discord | `DISCORD_BOT_TOKEN`, `DISCORD_PUBLIC_KEY`, `DISCORD_APPLICATION_ID` |
| GitHub | `GITHUB_TOKEN`, `GITHUB_WEBHOOK_SECRET`, `BOT_USERNAME` |
| State | `REDIS_URL` |

---

## Event Handlers

```typescript
// When someone @mentions the bot
bot.onNewMention(async (thread) => {
  await thread.subscribe(); // Listen to follow-ups
  await thread.post('Hello! Ask me anything.');
});

// When a message arrives in a subscribed thread
bot.onSubscribedMessage(async (thread, message) => {
  await thread.post(`You said: ${message.text}`);
});

// Reactions
bot.onReaction(async (thread, reaction) => {
  if (reaction.emoji === 'thumbsup') {
    await thread.post('Thanks!');
  }
});

// Slash commands
bot.onSlashCommand('/status', async (event) => {
  await event.channel.post('All systems operational!');
});

// Catch-all slash command
bot.onSlashCommand(async (event) => {
  await event.channel.post(`Unknown command: ${event.command}`);
});

// Button/action clicks
bot.onAction('escalate', async (event) => {
  await event.openModal(/* ... */);
});

// Modal form submissions
bot.onModalSubmit('form-id', async (event) => {
  const { description, priority } = event.values;
  // Process form data
});
```

### Thread & Channel API

```typescript
// Post messages
const sent = await thread.post('Hello');
await sent.edit('Updated message');
await sent.delete();
await sent.react('thumbsup');

// Ephemeral messages (visible only to one user — requires user parameter)
await thread.postEphemeral(userId, 'Only you see this', { fallbackToDM: true });

// Direct messages
const dm = await bot.openDM(userId);
await dm.post('Private message');

// Channel access
await thread.channel.post('Channel-wide message');
```

---

## Cards & Modals

### JSX Cards (cross-platform)

```tsx
import { Card, Text, Actions, Button, Fields, Field } from 'chat';

bot.onNewMention(async (thread) => {
  await thread.subscribe();
  await thread.post(
    <Card title="Support Ticket">
      <Fields>
        <Field title="Status" value="Open" />
        <Field title="Priority" value="High" />
      </Fields>
      <Text>How can I help you today?</Text>
      <Actions>
        <Button id="escalate">Escalate to Human</Button>
        <Button id="close">Close Ticket</Button>
      </Actions>
    </Card>
  );
});
```

Cards compile to platform-native formats: Slack Block Kit, Teams Adaptive Cards, Google Card format.

### Modals

```tsx
import { Modal, TextInput, Select, SelectOption } from 'chat';

bot.onAction('escalate', async (event) => {
  await event.openModal(
    <Modal callbackId="escalation-form" title="Escalate" submitLabel="Submit" closeLabel="Cancel">
      <TextInput id="description" label="Description" multiline />
      <Select id="priority" label="Priority">
        <SelectOption value="high">High</SelectOption>
        <SelectOption value="medium">Medium</SelectOption>
        <SelectOption value="low">Low</SelectOption>
      </Select>
    </Modal>
  );
});

bot.onModalSubmit('escalation-form', async (event) => {
  const { description, priority } = event.values;
  await event.thread.post(`Escalated: ${description} (${priority})`);
});
```

Modals supported on Slack. Not supported on Telegram.

---

## AI SDK Integration

Chat SDK's `post()` accepts AI SDK text streams for real-time AI responses:

```typescript
import { streamText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

bot.onSubscribedMessage(async (thread, message) => {
  const result = streamText({
    model: anthropic('claude-sonnet-4-5'),
    prompt: message.text,
  });
  await thread.post(result.toTextStream()); // Streams to chat platform
});
```

---

## Platform Adapters

### Multi-platform bot

```typescript
import { Chat } from 'chat';
import { createSlackAdapter } from '@chat-adapter/slack';
import { createDiscordAdapter } from '@chat-adapter/discord';
import { createTeamsAdapter } from '@chat-adapter/teams';
import { createGChatAdapter } from '@chat-adapter/gchat';
import { createTelegramAdapter } from '@chat-adapter/telegram';
import { createGitHubAdapter } from '@chat-adapter/github';
import { createRedisState } from '@chat-adapter/state-redis';

const bot = new Chat({
  userName: 'mybot',
  adapters: {
    slack: createSlackAdapter(),
    discord: createDiscordAdapter(),
    teams: createTeamsAdapter(),
    gchat: createGChatAdapter(),
    telegram: createTelegramAdapter(),
    github: createGitHubAdapter(),
  },
  state: createRedisState(),
});
```

### Discord note
Discord uses Gateway WebSocket (not webhooks). The adapter includes a built-in Gateway listener that connects to the WebSocket and forwards events to your webhook endpoint.

---

## Deployment

### Next.js

```typescript
// app/api/webhooks/[platform]/route.ts
import { bot } from '@/lib/bot';

export async function POST(request: Request, { params }: { params: { platform: string } }) {
  const handler = bot.webhooks[params.platform];
  if (!handler) return new Response('Unknown platform', { status: 404 });
  return handler(request, {
    waitUntil: (promise) => { /* Required on serverless */ },
  });
}
```

### Hono

```typescript
import { Hono } from 'hono';
import { bot } from './bot';

const app = new Hono();
app.post('/api/webhooks/:platform', async (c) => {
  const handler = bot.webhooks[c.req.param('platform')];
  return handler(c.req.raw, { waitUntil: c.executionCtx.waitUntil });
});
```

### Nuxt

```typescript
// server/api/webhooks/[platform].post.ts
import { bot } from '../lib/bot';

export default defineEventHandler(async (event) => {
  const platform = getRouterParam(event, 'platform');
  const handler = bot.webhooks[platform];
  return handler(toWebRequest(event));
});
```

### Official guides
- [Slack bot with Next.js](https://chat-sdk.dev/docs/guides/slack-nextjs)
- [Discord bot with Nuxt](https://chat-sdk.dev/docs/guides/discord-nuxt)
- [GitHub code review bot with Hono](https://chat-sdk.dev/docs/guides/code-review-hono)
