# Agentic Usage - OpenRouter SDK Documentation

## Overview

The OpenRouter SDK skill enables AI coding assistants to understand and work with OpenRouter's APIs. Installation is simple via command line.

## Installation

Run the following command in your project:

```bash
npx add-skill OpenRouterTeam/agent-skills
```

This installs the skill, teaching your AI assistant how to leverage the SDK effectively.

## Compatible Assistants

The skill supports multiple AI coding platforms including Claude Code, OpenCode, Cursor, GitHub Copilot, Codex, Amp, Roo Code, and Antigravity.

## Skill Capabilities

Once installed, your assistant gains knowledge of:

* SDK installation and configuration for TypeScript
* callModel API with type safety and streaming
* Chat Completions for conversational interfaces
* Embeddings for semantic search and RAG applications
* Error handling patterns and Result types
* Real-time streaming responses
* Function calling and tool implementation

## Code Examples

**Basic Model Call:**
```typescript
import { callModel } from '@openrouter/sdk';

const response = await callModel({
  model: 'anthropic/claude-sonnet-4',
  messages: [
    { role: 'user', content: 'Hello!' }
  ]
});
```

**Streaming Implementation:**
```typescript
const stream = await callModel({
  model: 'anthropic/claude-sonnet-4',
  messages: [{ role: 'user', content: 'Tell me a story' }],
  stream: true
});

for await (const chunk of stream) {
  process.stdout.write(chunk.choices[0]?.delta?.content ?? '');
}
```

## Updates & Manual Setup

Rerun the installation command for updates. Alternatively, manually add the skill file to `.skills/openrouter-sdk.md` via the GitHub repository.
