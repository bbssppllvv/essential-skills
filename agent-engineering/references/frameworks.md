# Agent Frameworks — Reference

A decision guide and code reference for TypeScript agent frameworks.

Evidence basis:
- [Vercel AI SDK Agents docs](https://ai-sdk.dev/docs/agents/overview)
- [Claude Agent SDK TypeScript docs](https://docs.claude.com/en/docs/agent-sdk/typescript)
- [OpenAI Agents SDK docs](https://openai.github.io/openai-agents-js/)
- [Mastra docs](https://mastra.ai/en/docs)
- [LangGraph JS docs](https://langchain-ai.github.io/langgraphjs/)
- [LlamaIndex TypeScript docs](https://ts.llamaindex.ai/)

Framework APIs change quickly. Use this file to choose a direction, then verify snippets against the framework's official docs.

## Decision Matrix

| Framework | Best For | Multi-Agent | DX |
|-----------|----------|-------------|-----|
| **Vercel AI SDK v6** | Web apps, Next.js, chat UIs, rapid prototyping | `stopWhen` + `prepareStep` | Highest for web |
| **Claude Agent SDK** | Production coding agents, Claude Code-like systems | Subagents with isolated context | High |
| **OpenAI Agents SDK** | Guardrails, voice agents, handoff patterns | Handoffs (agents as tools) | High |
| **Mastra** | TS-native DX, model routing, observational memory | Supervisor pattern | Highest overall |
| **LangGraph.js** | Complex stateful workflows, persistence | Graph-based state machines | Medium |
| **LlamaIndex.TS** | RAG-centric agents | Limited | High |
| **Raw Anthropic SDK** | Maximum control, custom protocols | Manual implementation | Medium |

**Guidance**: Start with Vercel AI SDK for web apps. Use Claude Agent SDK for coding agents. Use raw Anthropic SDK when you need fine-grained control. Consider Mastra for TS-native DX with multi-provider support.

---

## Vercel AI SDK v6

Primary framework for web-facing agents, Next.js chat UIs, streaming, and typed tool calling. Prefer AI SDK Core (`generateText`, `streamText`, `ToolLoopAgent`) when you want model/provider flexibility without building a raw loop.

### ToolLoopAgent (the main abstraction)

```typescript
import { ToolLoopAgent, Output, tool, stepCountIs } from "ai";
import { anthropic } from "@ai-sdk/anthropic";
import { z } from "zod";

const agent = new ToolLoopAgent({
  model: anthropic("claude-sonnet-4-6"),
  instructions: "You are a helpful assistant.",
  tools: {
    weather: tool({
      description: "Get weather for a location",
      inputSchema: z.object({ location: z.string() }),
      execute: async ({ location }) => ({ temperature: 72, conditions: "sunny" }),
    }),
  },
  stopWhen: stepCountIs(10),
  output: Output.object({
    schema: z.object({ summary: z.string(), temperature: z.number() }),
  }),
  onStepFinish: async ({ stepNumber, usage }) => {
    console.log(`Step ${stepNumber}: ${usage.totalTokens} tokens`);
  },
});

// Generate
const { text, output, steps, totalUsage } = await agent.generate({
  prompt: "What's the weather in SF?",
});

// Stream
const result = await agent.stream({ prompt: "What's the weather in SF?" });
for await (const chunk of result.textStream) process.stdout.write(chunk);
```

### Key Features

**`prepareStep`** — dynamic per-step configuration:
```typescript
prepareStep: async ({ stepNumber, messages }) => {
  if (stepNumber === 0) return { model: cheapModel, toolChoice: { type: "tool", toolName: "search" } };
  if (messages.length > 20) return { messages: messages.slice(-10) }; // trim context
  return {};
},
```

**`needsApproval`** — human-in-the-loop:
```typescript
tool({
  description: "Run shell command",
  inputSchema: z.object({ command: z.string() }),
  needsApproval: async ({ command }) => command.includes("rm"),
  execute: async ({ command }) => execSync(command, { encoding: "utf-8" }),
})
```

**`toModelOutput`** — separate what execute returns from what the model sees:
```typescript
tool({
  execute: async ({ location }) => ({ temperature: 72, humidity: 45, forecast: [/* big */] }),
  toModelOutput: async ({ input, output }) => ({
    type: "text",
    value: `Weather in ${input.location}: ${output.temperature}F`,
  }),
})
```

**Structured output** with tool calling:
```typescript
output: Output.object({ schema: z.object({ summary: z.string() }) })
// Also: Output.array(), Output.choice(), Output.json(), Output.text()
```

### Next.js Integration

```typescript
// app/api/chat/route.ts
import { createAgentUIStreamResponse } from "ai";

export async function POST(req: Request) {
  const { messages } = await req.json();
  return createAgentUIStreamResponse({ agent: myAgent, uiMessages: messages });
}

// app/page.tsx — client
import { useChat } from "@ai-sdk/react";
const { messages, sendMessage, status } = useChat();
```

### MCP Integration
```typescript
import { createMCPClient } from "@ai-sdk/mcp";
const mcpClient = await createMCPClient({ transport: { type: "http", url: "https://..." } });
const tools = await mcpClient.tools();
```

---

## Claude Agent SDK

The same loop that powers Claude Code. MIT licensed.

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Refactor the auth module",
  options: {
    allowedTools: ["Read", "Edit", "Bash", "Glob", "Grep"],
    permissionMode: "acceptEdits",
    hooks: { PostToolUse: [{ matcher: "Edit|Write", hooks: [auditLogger] }] },
    agents: {
      "code-reviewer": {
        description: "Reviews code quality",
        tools: ["Read", "Glob", "Grep"],
      },
    },
    mcpServers: {
      playwright: { command: "npx", args: ["@playwright/mcp@latest"] },
    },
  },
})) {
  console.log(message);
}
```

Built-in tools: Read, Write, Edit, Bash, Monitor, Glob, Grep, WebSearch, WebFetch, AskUserQuestion.

---

## OpenAI Agents SDK

Production-ready for OpenAI-first text/voice agents with handoffs, guardrails, tracing, sessions, and hosted tools. It can use custom model providers, but OpenAI-hosted tools and Responses-specific features are strongest on OpenAI models. The JS/TS repo requires Node.js 22+.

```typescript
import { Agent, run } from "@openai/agents";

const agent = new Agent({
  name: "Support",
  instructions: "You are a helpful support agent.",
  tools: [weatherTool],
});

const result = await run(agent, "What's the weather?");
```

**Handoff pattern** — agents as callable tools:
```typescript
const specialist = new Agent({ name: "Billing", instructions: "Handle billing issues." });
const router = new Agent({
  name: "Router",
  instructions: "Route queries to specialists.",
  handoffs: [specialist], // specialist appears as a tool
});
```

**Per-tool guardrails**:
```typescript
const tool: FunctionTool = {
  name: "delete_account",
  needsApproval: async (params) => params.permanent === true,
  inputGuardrails: [validateUserIdentity],
  outputGuardrails: [sanitizePII],
  timeoutMs: 5000,
};
```

---

## Mastra

TS-native framework for model routing, memory, workflows, and multi-provider agent apps. Verify current provider counts and memory APIs before copying examples; the framework changes quickly.

```typescript
import { Agent } from "@mastra/core/agent";
import { createTool } from "@mastra/core/tools";
import { Memory } from "@mastra/memory";
import { z } from "zod";

const weatherTool = createTool({
  id: "weather",
  description: "Get weather",
  inputSchema: z.object({ location: z.string() }),
  outputSchema: z.object({ weather: z.string() }),
  execute: async ({ location }) => ({ weather: "72F, sunny" }),
});

const agent = new Agent({
  id: "assistant",
  model: "anthropic/claude-sonnet-4-6",
  tools: { weather: weatherTool },
  memory: new Memory({ options: { lastMessages: 20, observationalMemory: true } }),
});

// Supervisor pattern
const supervisor = new Agent({
  id: "supervisor",
  agents: { writer, researcher }, // sub-agents as tools
});
```

**Observational memory**: agent autonomously notices and remembers patterns.
**Cross-agent memory sharing** via resource scoping.

---

## LangGraph.js

Graph-based state management. Most architecturally rigorous.

```typescript
import { createReactAgent } from "@langgraphjs/toolkit";
import { ChatAnthropic } from "@langchain/anthropic";

const agent = createReactAgent({
  llm: new ChatAnthropic({ model: "claude-sonnet-4-6" }),
  tools: [weatherTool],
});

// Streaming
const stream = await agent.stream(
  { messages: [{ role: "user", content: "Weather in SF?" }] },
  { streamMode: "messages" }
);
for await (const [message, metadata] of stream) {
  if (metadata.langgraph_node === "agent") process.stdout.write(String(message.content));
}
```

**Custom state with reducers** — track metadata across turns:
```typescript
const AgentState = Annotation.Root({
  messages: Annotation<BaseMessage[]>({ reducer: messagesStateReducer }),
  toolCallCount: Annotation<number>({ reducer: (cur, upd) => (cur ?? 0) + upd }),
});
```

---

## LlamaIndex.TS

Best RAG integration. Clean API.

```typescript
import { agent, agentStreamEvent, tool } from "llamaindex";
import { z } from "zod";

const myAgent = agent({
  tools: [
    tool({
      name: "search",
      description: "Search knowledge base",
      parameters: z.object({ query: z.string() }),
      execute: ({ query }) => searchVectorStore(query),
    }),
  ],
});

const events = myAgent.runStream("Find info about auth patterns");
for await (const event of events) {
  if (agentStreamEvent.include(event)) process.stdout.write(event.data.delta);
}
```

---

## Raw Anthropic SDK (Build From Scratch)

Maximum control. The minimal agent is ~50 lines of real logic:

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const conversation: Anthropic.MessageParam[] = [];

// Outer loop: user turns
while (true) {
  const userInput = await ask("> ");
  conversation.push({ role: "user", content: userInput });

  // Inner loop: agent turns
  let response = await client.messages.create({
    model: "claude-sonnet-4-6", max_tokens: 4096, tools, messages: conversation,
  });
  conversation.push({ role: "assistant", content: response.content });

  while (response.stop_reason === "tool_use") {
    const results = response.content
      .filter((b): b is Anthropic.ToolUseBlock => b.type === "tool_use")
      .map((tc) => ({
        type: "tool_result" as const,
        tool_use_id: tc.id,
        content: String(executeTool(tc.name, tc.input)),
      }));
    conversation.push({ role: "user", content: results });
    response = await client.messages.create({
      model: "claude-sonnet-4-6", max_tokens: 4096, tools, messages: conversation,
    });
    conversation.push({ role: "assistant", content: response.content });
  }
}
```

---

## Production Autonomy Levels

| Level | Pattern | Production Readiness |
|-------|---------|---------------------|
| 1 | Prompt Chaining | Proven, low risk |
| 2 | Workflows with Branching | Proven, the sweet spot |
| 3 | Tool-Using Agents (ReAct) | Proven with guardrails |
| 4 | Multi-Agent Systems | Painful for production |
| 5 | Learning Agents | Experimental only |

**Default stance**: Levels 2-3 are the production sweet spot. Level 4 can be valuable for broad research or independent parallel work, but it increases token spend, coordination complexity, and eval difficulty.

---

## Protocol Stack (2025-2026)

- **MCP** (Model Context Protocol): Agent-to-tool communication. "USB for AI tools."
- **A2A** (Agent-to-Agent): emerging inter-agent collaboration protocol; verify adoption and SDK maturity before using in production.
- **AG-UI** (Agent-User Interaction): emerging UI-agent event protocol; useful to know, but not yet as universal as MCP.
