# Agents, Tools & MCP Reference

## Table of Contents
1. [Tool Calling](#tool-calling)
2. [Multi-Step Agents](#multi-step-agents)
3. [ToolLoopAgent](#toolloopagent)
4. [MCP Integration](#mcp-integration)
5. [Tool Approval](#tool-approval)
6. [Subagents & Multi-Agent](#subagents--multi-agent)
7. [Memory & State Management](#memory--state-management)
8. [MCP Advanced Features](#mcp-advanced-features)
9. [Building MCP Servers](#building-mcp-servers)
10. [Workflow Design Patterns](#workflow-design-patterns)

---

## Tool Calling

### Defining tools

```typescript
import { tool } from 'ai';
import { z } from 'zod';

const weatherTool = tool({
  description: 'Get current weather for a location',
  inputSchema: z.object({  // MUST use inputSchema, NOT parameters
    city: z.string().describe('City name'),
    units: z.enum(['celsius', 'fahrenheit']).default('celsius'),
  }),
  execute: async ({ city, units }) => {
    const data = await fetchWeather(city, units);
    return { temperature: data.temp, condition: data.condition };
  },
});
```

**Critical**: In AI SDK v6, use `inputSchema` (not `parameters`). The `tool()` function is required — do not pass raw objects.

### Using tools with generateText/streamText

```typescript
import { generateText, tool, stepCountIs } from 'ai';
import { z } from 'zod';

const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  tools: {
    getWeather: tool({
      description: 'Get weather',
      inputSchema: z.object({ city: z.string() }),
      execute: async ({ city }) => ({ temp: 72, condition: 'sunny' }),
    }),
    searchWeb: tool({
      description: 'Search the web',
      inputSchema: z.object({ query: z.string() }),
      execute: async ({ query }) => ({ results: ['...'] }),
    }),
  },
  stopWhen: stepCountIs(5), // Allow multiple tool-use rounds
  messages: [{ role: 'user', content: 'What is the weather in Paris?' }],
});

// Access tool calls and results
console.log(result.toolCalls);    // Array of tool invocations
console.log(result.toolResults);  // Array of tool outputs
console.log(result.steps);       // Array of all steps taken
```

### Client-side tool execution

```typescript
const { messages } = useChat({
  sendAutomaticallyWhen: ({ messages }) => {
    // Auto-send when last message has tool results (replaces removed maxSteps)
    const lastMsg = messages[messages.length - 1];
    return lastMsg?.parts.some(p => p.type.startsWith('tool-'));
  },
  onToolCall: async ({ toolCall }) => {
    // Execute tool on the client
    if (toolCall.toolName === 'getLocation') {
      return navigator.geolocation.getCurrentPosition();
    }
  },
});
```

### Tool output schema

```typescript
const myTool = tool({
  description: 'Analyze sentiment',
  inputSchema: z.object({ text: z.string() }),
  outputSchema: z.object({
    sentiment: z.enum(['positive', 'negative', 'neutral']),
    confidence: z.number(),
  }),
  execute: async ({ text }) => ({
    sentiment: 'positive',
    confidence: 0.95,
  }),
});
```

---

## Multi-Step Agents

Use `stopWhen` with `stepCountIs()` to allow the model to call tools iteratively until it has enough information or hits the step limit.

```typescript
import { streamText, stepCountIs, hasToolCall } from 'ai';

const result = await streamText({
  model: anthropic('claude-sonnet-4-5'),
  system: 'You are a research assistant. Use tools to find information.',
  tools: {
    searchWeb: tool({ /* ... */ }),
    readPage: tool({ /* ... */ }),
    calculateStats: tool({ /* ... */ }),
  },
  stopWhen: stepCountIs(10), // Up to 10 tool-use rounds (replaces removed maxSteps)
  messages,
  onStepFinish: ({ text, toolCalls, toolResults, stepType }) => {
    console.log(`Step completed: ${stepType}`);
  },
});
```

### Stopping conditions

```typescript
import { stepCountIs, hasToolCall } from 'ai';

// Stop after N steps
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  tools: { /* ... */ },
  stopWhen: stepCountIs(5),
  messages,
});

// Stop when a specific tool is called
const result2 = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  tools: { /* ... */ },
  stopWhen: hasToolCall('finalAnswer'), // Stop when this tool is called
  messages,
});
```

---

## ToolLoopAgent

The `ToolLoopAgent` class (AI SDK v6) provides a high-level abstraction for agentic applications.

```typescript
import { ToolLoopAgent, createAgentUIStreamResponse, stepCountIs } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const agent = new ToolLoopAgent({
  model: anthropic('claude-sonnet-4-5'),
  instructions: 'You are a helpful coding assistant.', // NOT `system` — agents use `instructions`
  tools: {
    readFile: tool({ /* ... */ }),
    writeFile: tool({ /* ... */ }),
    runCommand: tool({ /* ... */ }),
  },
  stopWhen: stepCountIs(20), // Default is stepCountIs(20) for agents
});
```

### Using with API routes

```typescript
// app/api/agent/route.ts
// createAgentUIStreamResponse is a standalone function, NOT a method on the agent
export async function POST(req: Request) {
  const { messages } = await req.json();
  return createAgentUIStreamResponse({ agent, uiMessages: messages });
}
```

### Client-side

```tsx
const { messages, sendMessage } = useChat({
  api: '/api/agent',
});
```

---

## MCP Integration

Model Context Protocol enables connecting to external tool servers.

```typescript
import { createMCPClient } from '@ai-sdk/mcp';

// HTTP transport (remote server — preferred over deprecated SSE)
const mcpClient = await createMCPClient({
  transport: {
    type: 'http',
    url: 'https://mcp-server.example.com/mcp',
  },
});

// Stdio transport (local process)
const mcpClient = await createMCPClient({
  transport: {
    type: 'stdio',
    command: 'npx',
    args: ['-y', '@modelcontextprotocol/server-filesystem', '/tmp'],
  },
});

// Use MCP tools with generateText/streamText
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  tools: await mcpClient.tools(),
  stopWhen: stepCountIs(10),
  messages,
});

// Clean up
await mcpClient.close();
```

### Combining MCP tools with local tools

```typescript
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  tools: {
    ...await mcpClient.tools(),  // MCP tools
    myLocalTool: tool({ /* ... */ }),  // Local tools
  },
  stopWhen: stepCountIs(10),
  messages,
});
```

### Packages needed

```bash
npm install @ai-sdk/mcp @modelcontextprotocol/sdk
```

---

## Tool Approval

Require user confirmation before executing dangerous tools.

### Server-side

In v6, tool approval uses the `needsApproval` property directly on the tool (not on streamText):

```typescript
const result = streamText({
  model: anthropic('claude-sonnet-4-5'),
  tools: {
    deleteFile: tool({
      description: 'Delete a file',
      inputSchema: z.object({ path: z.string() }),
      needsApproval: true, // v6 way — on the tool itself, NOT on streamText
      execute: async ({ path }) => fs.unlinkSync(path),
    }),
    readFile: tool({
      description: 'Read a file',
      inputSchema: z.object({ path: z.string() }),
      execute: async ({ path }) => fs.readFileSync(path, 'utf-8'),
    }),
  },
  stopWhen: stepCountIs(5),
  messages,
});

// needsApproval can also be a function for conditional approval:
const conditionalTool = tool({
  description: 'Write to file',
  inputSchema: z.object({ path: z.string(), content: z.string() }),
  needsApproval: ({ args }) => args.path.includes('/system/'), // Only approve system files
  execute: async ({ path, content }) => fs.writeFileSync(path, content),
});
```

### Client-side with AI Elements Confirmation

```tsx
const { addToolApprovalResponse } = useChat();

// In message parts rendering — tool parts use dynamic types:
// part.type will be 'tool-deleteFile', 'tool-readFile', etc.
  if (part.state === 'approval-requested') {
    return (
      <Confirmation>
        <ConfirmationRequest>
          Allow {part.toolName}({JSON.stringify(part.args)})?
          <ConfirmationActions>
            <ConfirmationAction
              onClick={() => addToolApprovalResponse({ id: part.approval.id, approved: true })}
            >
              Allow
            </ConfirmationAction>
            <ConfirmationAction
              onClick={() => addToolApprovalResponse({ id: part.approval.id, approved: false })}
            >
              Deny
            </ConfirmationAction>
          </ConfirmationActions>
        </ConfirmationRequest>
        <ConfirmationAccepted>Approved</ConfirmationAccepted>
        <ConfirmationRejected>Denied</ConfirmationRejected>
      </Confirmation>
    );
  }
  return <Tool part={part} />;
```

---

## Subagents & Multi-Agent

Subagents are autonomous agents invoked through tools by a parent agent, operating with isolated context windows.

### Why subagents

- **Context offloading**: Subagent consumes hundreds of thousands of tokens exploring, returns only a focused summary
- **Parallelization**: Multiple subagents research different areas simultaneously
- **Specialization**: Separate subagents with distinct tools and instructions

### Implementation

```typescript
const researchAgent = new ToolLoopAgent({
  model: anthropic('claude-sonnet-4-5'),
  instructions: 'You are a research specialist.',
  tools: { searchWeb: tool({ /* ... */ }), readPage: tool({ /* ... */ }) },
});

const mainAgent = new ToolLoopAgent({
  model: anthropic('claude-sonnet-4-5'),
  instructions: 'You orchestrate research tasks.',
  tools: {
    research: tool({
      description: 'Delegate a research task to a specialist',
      inputSchema: z.object({ task: z.string() }),
      execute: async ({ task }, { abortSignal }) => {
        const result = await researchAgent.generate({ prompt: task, abortSignal });
        return result.text; // Only the summary reaches the main agent
      },
    }),
  },
});
```

### Context control with toModelOutput

Separate what the user sees from what the model consumes:

```typescript
const screenshotTool = tool({
  description: 'Take a screenshot',
  inputSchema: z.object({ url: z.string() }),
  execute: async ({ url }) => {
    const screenshot = await takeScreenshot(url);
    return { screenshot, description: 'Screenshot of ' + url };
  },
  toModelOutput: ({ output }) => output.description, // Model sees only text summary
});
```

When to use subagents: when tasks require exploring large token volumes or context would grow unmanageable. Skip for simple, focused work where sequential processing suffices.

---

## Memory & State Management

### Provider memory tools (Anthropic)

```typescript
// Anthropic's built-in memory tool
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  tools: { memory: anthropic.tools.memory() },
  messages,
});
```

### External memory providers

| Provider | Description |
|----------|-------------|
| **Letta** | Core and archival memory management |
| **Mem0** | Extracts memories from conversations |
| **Supermemory** | Semantic search for automatic storage/retrieval |
| **Hindsight** | Five tools: retain, recall, reflect, etc. |

### Custom memory tools

```typescript
const rememberTool = tool({
  description: 'Save information for later recall',
  inputSchema: z.object({ key: z.string(), value: z.string() }),
  execute: async ({ key, value }) => {
    await db.memories.upsert({ key, value });
    return 'Saved';
  },
});

const recallTool = tool({
  description: 'Recall saved information',
  inputSchema: z.object({ query: z.string() }),
  execute: async ({ query }) => {
    const memories = await db.memories.search(query);
    return memories;
  },
});
```

---

## MCP Advanced Features

### OAuth Authentication

```typescript
const mcpClient = await createMCPClient({
  transport: {
    type: 'http',
    url: 'https://mcp-server.example.com/mcp',
    auth: {
      type: 'oauth',
      clientId: 'your-client-id',
      // PKCE flow, token refresh, dynamic client registration supported
    },
  },
});
```

### Resources

```typescript
const resources = await mcpClient.listResources();
const content = await mcpClient.readResource('resource://docs/api-reference');

const templates = await mcpClient.listResourceTemplates();
```

### Prompts

```typescript
const prompts = await mcpClient.experimental_listPrompts();
const prompt = await mcpClient.experimental_getPrompt('summarize', { text: 'Hello world' });
```

### Elicitation

MCP servers can request user input during tool execution. The server sends an elicitation request, and the client responds with user-provided data.

### Typed tool outputs

```typescript
const mcpClient = await createMCPClient({
  transport: { type: 'http', url: 'https://server.example.com' },
  toolDefinitions: {
    getWeather: {
      outputSchema: z.object({ temp: z.number(), condition: z.string() }),
    },
  },
});
```

---

## Building MCP Servers

### Using vercel/mcp-handler

```bash
npm install @vercel/mcp-handler @modelcontextprotocol/sdk
```

```typescript
// app/api/mcp/route.ts
import { createMcpHandler } from '@vercel/mcp-handler';
import { z } from 'zod';

const handler = createMcpHandler(
  (server) => {
    server.registerTool('roll_dice', {
      title: 'Roll Dice',
      inputSchema: { sides: z.number().int().min(2) },
    }, async ({ sides }) => ({
      content: [{ type: 'text', text: `Rolled ${Math.floor(Math.random() * sides) + 1}` }],
    }));
  },
  {},
  { basePath: '/api/mcp', maxDuration: 60 }
);

export { handler as GET, handler as POST };
```

Supports: Streamable HTTP, SSE transports, Redis for SSE resumability, OAuth authorization.

---

## Workflow Design Patterns

Five patterns for composing LLM calls (from AI SDK docs, separate from the durable Workflow DevKit):

### 1. Sequential Processing (Chains)
Linear step execution — output feeds to next step.

### 2. Routing
Model classifies input, routes to specialized handler:
```typescript
const { output } = await generateText({
  model, prompt: userInput,
  output: Output.choice({ options: ['technical', 'billing', 'general'] }),
});
// Route to specialized handler based on classification
```

### 3. Parallel Processing
Independent concurrent subtasks:
```typescript
const [summary, sentiment, keywords] = await Promise.all([
  generateText({ model, prompt: 'Summarize: ' + text }),
  generateText({ model, prompt: 'Analyze sentiment: ' + text }),
  generateText({ model, prompt: 'Extract keywords: ' + text }),
]);
```

### 4. Orchestrator-Worker
Central model delegates to specialized workers.

### 5. Evaluator-Optimizer
Iterative generation with quality gates (max N iterations).

---

## Claim Deployments

Enable AI agents to create full-stack applications, deploy them to Vercel, and transfer ownership to users. This pattern supports building "vibe coding" tools where an AI generates and deploys a project, then hands it to the end user.

### How It Works

1. **AI agent creates a project** on your Vercel team (using Vercel REST API)
2. **Agent deploys code** to the project
3. **User claims the deployment** — ownership transfers from your team to theirs via a claim token

### Setup

```typescript
// 1. Create deployment as your team
const deployment = await fetch('https://api.vercel.com/v13/deployments', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${process.env.VERCEL_TOKEN}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    name: 'user-generated-app',
    target: 'production',
    files: deploymentFiles,
    projectSettings: { framework: 'nextjs' },
  }),
});

// 2. Generate a claim token for the deployment
const claim = await fetch(
  `https://api.vercel.com/v1/claim/deployment?deploymentId=${deployment.id}`,
  {
    method: 'POST',
    headers: { Authorization: `Bearer ${process.env.VERCEL_TOKEN}` },
  }
);
const { claimToken } = await claim.json();

// 3. Give the user a claim URL
const claimUrl = `https://vercel.com/claim/${claimToken}`;
// User visits this URL → project transfers to their Vercel account
```

### Use Cases

- **AI app builders**: Users describe an app, AI generates it, deploys to Vercel, user claims it
- **Template generators**: Dynamic template creation with pre-configured settings
- **Code education**: AI generates project exercises, students claim and iterate

### Integration with Sandbox

Combine with Vercel Sandbox for a build-then-deploy pattern:

```typescript
// 1. AI builds in sandbox (safe, isolated)
const sandbox = await Sandbox.create({ runtime: 'node24' });
await sandbox.writeFiles(generatedFiles);
await sandbox.runCommand('npm', ['install']);
await sandbox.runCommand('npm', ['run', 'build']);

// 2. Export and deploy
const buildOutput = await sandbox.readFileToBuffer({ path: 'out/' });
// Deploy via Vercel API...

// 3. Generate claim URL for user
```
