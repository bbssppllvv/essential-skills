# Harness Patterns — Deep Dive

Production-grade scaffolding for AI agents. The harness is everything around the LLM: how you run the loop, manage tokens, handle failures, and surface progress to the user.

Evidence basis:
- [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic: Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [OpenAI: Unrolling the Codex agent loop](https://openai.com/index/unrolling-the-codex-agent-loop/)
- [Vercel AI SDK streaming docs](https://ai-sdk.dev/docs/foundations/streaming)
- [OpenTelemetry](https://opentelemetry.io/)

Operational patterns here are stable; API names and provider billing fields are volatile and must be checked in official docs.

## Table of Contents
1. [Streaming](#streaming)
2. [Context Management](#context-management)
3. [Safety and Permissions](#safety-and-permissions)
4. [State Persistence](#state-persistence)
5. [Prompt Caching](#prompt-caching)
6. [Error Recovery and Retries](#error-recovery-and-retries)
7. [Cost Controls](#cost-controls)
8. [Observability](#observability)
9. [Deployment Patterns](#deployment-patterns)

---

## Streaming

### Why Streaming Matters

An agent that takes 30 seconds of silence before responding feels broken. Streaming shows the user that work is happening — token by token for text, step by step for tool calls.

### Streaming with the Anthropic SDK (Node.js)

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const stream = await client.messages.stream({
  model: "claude-sonnet-4-6",
  messages: conversation,
  tools,
  max_tokens: 4096,
});

for await (const event of stream) {
  switch (event.type) {
    case "content_block_start":
      if (event.content_block.type === "text") {
        process.stdout.write(""); // text block starting
      } else if (event.content_block.type === "tool_use") {
        console.log(`\nCalling: ${event.content_block.name}`);
      }
      break;

    case "content_block_delta":
      if (event.delta.type === "text_delta") {
        process.stdout.write(event.delta.text); // stream text tokens
      } else if (event.delta.type === "input_json_delta") {
        // tool input being constructed — can show progress
      }
      break;

    case "message_stop":
      // Full response ready — check for tool calls
      break;
  }
}

const finalMessage = await stream.finalMessage();
```

### Streaming with Vercel AI SDK

If you're building a web-based agent, the Vercel AI SDK handles streaming elegantly:

```typescript
import { streamText, stepCountIs } from "ai";
import { anthropic } from "@ai-sdk/anthropic";

const result = streamText({
  model: anthropic("claude-sonnet-4-6"),
  messages: conversation,
  tools: myTools,
  stopWhen: stepCountIs(10), // allow up to 10 tool-call rounds
  onStepFinish: ({ toolCalls, toolResults }) => {
    // Called after each tool execution round
    console.log("Tools called:", toolCalls);
  },
});

// Stream to the client via SSE
return result.toUIMessageStreamResponse();
```

On the client side:
```typescript
import { useChat } from "@ai-sdk/react";

function Chat() {
  const { messages, input, handleInputChange, handleSubmit } = useChat({
    api: "/api/chat",
  });
  // messages automatically update as tokens stream in
}
```

### Server-Sent Events (custom)

For custom streaming without a framework:

```typescript
// Server (Next.js API route)
export async function POST(req: Request) {
  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      const send = (event: string, data: any) => {
        controller.enqueue(
          encoder.encode(`event: ${event}\ndata: ${JSON.stringify(data)}\n\n`)
        );
      };

      // Run agent loop, streaming events
      for await (const event of runAgent(messages)) {
        send(event.type, event.data);
      }
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      Connection: "keep-alive",
    },
  });
}
```

---

## Context Management

### Token Counting

Before compaction, you need to know how many tokens you're using:

```typescript
// Rough estimation (good enough for budgeting)
function estimateTokens(text: string): number {
  return Math.ceil(text.length / 4); // ~4 chars per token for English
}

// Precise counting — use Anthropic's token counting API
const count = await client.messages.countTokens({
  model: "claude-sonnet-4-6",
  messages: conversation,
  tools,
  system: systemPrompt,
});
console.log(`Input tokens: ${count.input_tokens}`);
```

### Compaction Implementation

```typescript
const MAX_CONTEXT = 100_000; // tokens
const COMPACTION_THRESHOLD = 0.8; // compact at 80% full

async function maybeCompact(conversation: Message[]): Promise<Message[]> {
  const tokens = await countTokens(conversation);
  if (tokens < MAX_CONTEXT * COMPACTION_THRESHOLD) {
    return conversation; // not yet needed
  }

  // Keep system prompt + last N messages verbatim
  const recentCount = 6; // last 3 turns
  const recent = conversation.slice(-recentCount);
  const old = conversation.slice(0, -recentCount);

  // Ask the LLM to summarize older context
  const summary = await client.messages.create({
    model: "claude-haiku-4-5-20251001", // cheap model for summarization
    messages: [
      {
        role: "user",
        content: `Summarize this conversation context concisely. Preserve: file paths mentioned, decisions made, errors encountered, current task state.\n\n${formatMessages(old)}`,
      },
    ],
    max_tokens: 2000,
  });

  return [
    {
      role: "user",
      content: `[Previous context summary]\n${summary.content[0].text}`,
    },
    { role: "assistant", content: "Understood. I have the context." },
    ...recent,
  ];
}
```

### Tool Result Truncation

```typescript
function truncateToolResult(result: string, maxChars = 10_000): string {
  if (result.length <= maxChars) return result;

  const lines = result.split("\n");
  const totalLines = lines.length;
  const keepLines = Math.floor(maxChars / 80); // rough estimate

  return [
    `[Truncated: showing ${keepLines} of ${totalLines} lines]`,
    ...lines.slice(0, keepLines),
    `\n... ${totalLines - keepLines} more lines. Use offset/limit to read specific ranges.`,
  ].join("\n");
}
```

---

## Safety and Permissions

### Permission System

```typescript
type PermissionLevel = "always_allow" | "ask" | "always_deny";

interface PermissionRule {
  tool: string;
  pattern?: RegExp; // optional — match against args
  level: PermissionLevel;
}

const DEFAULT_PERMISSIONS: PermissionRule[] = [
  // Reads are side-effect-free, but still scope them to approved workspaces
  // and block/redact secrets such as .env files, private keys, and credentials.
  { tool: "read_file", level: "always_allow" },
  { tool: "glob", level: "always_allow" },
  { tool: "grep", level: "always_allow" },

  // Writes need approval
  { tool: "edit_file", level: "ask" },
  { tool: "write_file", level: "ask" },

  // Shell is filtered
  { tool: "bash", pattern: /^(ls|cat|echo|pwd|git status|git log|git diff)/, level: "always_allow" },
  { tool: "bash", pattern: /rm\s+-rf|git\s+push.*--force|DROP\s+TABLE/i, level: "always_deny" },
  { tool: "bash", level: "ask" }, // everything else — ask

  // Destructive actions always need approval
  { tool: "delete_file", level: "ask" },
];

async function checkPermission(
  toolName: string,
  args: Record<string, any>,
  rules: PermissionRule[]
): Promise<boolean> {
  const argsStr = JSON.stringify(args);

  for (const rule of rules) {
    if (rule.tool !== toolName) continue;
    if (rule.pattern && !rule.pattern.test(argsStr)) continue;

    switch (rule.level) {
      case "always_allow":
        return true;
      case "always_deny":
        return false;
      case "ask":
        return await promptUser(`Allow ${toolName}(${argsStr})?`);
    }
  }

  // Default: ask
  return await promptUser(`Allow ${toolName}(${argsStr})?`);
}
```

### Sandbox Patterns for Web Agents

If your agent runs on a server (e.g., Next.js API route), you can't prompt the user interactively. Options:

1. **Pre-approved tool list** — only allow safe tools in server-side agents
2. **Approval queue** — agent proposes actions, user approves via UI before execution
3. **Container isolation** — run agent tools in a Docker container or Firecracker VM (Vercel Sandbox)

---

## Brain / Hands / Session Decoupling

For long-running or cloud-scale agents, decouple three components that are typically bundled together:

| Component | Role | Key property |
|-----------|------|-------------|
| **Brain** (harness) | Calls the LLM, routes tool calls | **Stateless** — can be killed and restarted |
| **Hands** (sandbox) | Executes code and tools | **Replaceable** — uniform tool interface |
| **Session** | Stores all agent activity | **Append-only event log** — single source of truth |

The session log lives *outside* the LLM's context window. The harness queries it via `getEvents()` with positional slices, loading only what the model needs rather than bloating the context window.

**Recovery pattern**: when a harness container dies mid-task, a new harness calls `wake(sessionId)`, retrieves the event log, and resumes from the last recorded event. The agent continues without losing work.

**Security boundary**: credentials never reach the sandbox where agent-generated code runs. Authentication tokens are bundled at initialization (git repos) or held in external vaults (MCP tools), with a proxy handling the exchange. This prevents a compromised code-execution environment from accessing production secrets.

**Performance benefit**: lazy initialization lets harnesses start immediately without waiting for sandbox provisioning. Anthropic reported ~60% p50 and >90% p95 latency improvement after decoupling.

This pattern turns fragile "pets" (coupled, irreplaceable containers) into replaceable "cattle" — any harness failure becomes a recoverable tool-call error, not a lost session.

Source: [Anthropic: Scaling Managed Agents — Decoupling the brain from the hands (Apr 2026)](https://www.anthropic.com/engineering/managed-agents)

---

## State Persistence

### Checkpoint Pattern

Save conversation state so sessions can resume:

```typescript
interface AgentCheckpoint {
  id: string;
  conversation: Message[];
  toolState: Record<string, any>; // e.g., current working directory
  createdAt: string;
}

async function saveCheckpoint(state: AgentCheckpoint) {
  // Could be file, database, KV store
  await kv.set(`checkpoint:${state.id}`, JSON.stringify(state));
}

async function loadCheckpoint(id: string): Promise<AgentCheckpoint | null> {
  const data = await kv.get(`checkpoint:${id}`);
  return data ? JSON.parse(data as string) : null;
}

// In the agent loop:
async function agentLoop(sessionId: string, userMessage: string) {
  let state = await loadCheckpoint(sessionId);
  if (!state) {
    state = { id: sessionId, conversation: [], toolState: {}, createdAt: new Date().toISOString() };
  }

  state.conversation.push({ role: "user", content: userMessage });

  while (true) {
    const response = await callLLM(state.conversation);
    state.conversation.push({ role: "assistant", content: response.content });

    const toolCalls = extractToolCalls(response);
    if (!toolCalls.length) break;

    // Execute tools and append results...
    await saveCheckpoint(state); // Save after each tool execution
  }

  await saveCheckpoint(state); // Save final state
  return getLastTextResponse(state.conversation);
}
```

### Durable Workflow State

For long-running agents, save more than the chat transcript:
- Current plan and completed steps.
- Pending approvals.
- Tool-specific state and cursor positions.
- Artifact pointers.
- Cost and token budget usage.
- Checkpoint IDs for resumable workflows.
- Pending writes that completed before a failure, so resume does not repeat side effects.

Use a stable `thread_id` or session ID for every checkpointed run. Make resume explicit: the new harness process should read durable state, restore only safe permissions, and continue from the last checkpoint instead of asking the model to reconstruct work from memory.

### Request and Session Isolation

If the harness serves multiple users, browser tabs, MCP clients, or sub-agents concurrently, request-scoped context is a correctness boundary.

Isolate:
- auth/session identity
- logs and notifications
- temporary tool results
- skill/tool activation state
- MCP resource subscriptions
- approval prompts
- checkpoint thread IDs

Test this directly: run two concurrent clients with different log levels, permissions, or resources and prove that neither receives the other's events.

### Checkpoint Storage Safety

Checkpoint and cache stores can hold prompts, tool outputs, secrets accidentally returned by tools, and user data. Treat them as high-sensitivity data stores.

Controls:
- Prefer JSON or strict allowlisted serializers for persisted agent state.
- Avoid unrestricted `pickle`, dynamic imports, or broad module allowlists when data can be written by anything outside the trusted process.
- Encrypt sensitive checkpoints when database operators or adjacent tenants are outside the trust boundary.
- Set retention and deletion policies; do not keep conversation history indefinitely by default.
- Validate remote workflow responses before merging them into local state.

---

## Prompt Caching

Usually the highest-ROI optimization for agents. System prompts and tool definitions are sent with every request; provider prompt caching can make cache reads much cheaper than normal input tokens when the stable prefix is identical.

### Stable Prefix, Volatile Suffix

Design the context builder for cache reuse:

```text
stable prefix:
  tool definitions in deterministic order
  static system/developer instructions
  stable scoped instructions or skill index
  stable schemas and output contracts
  reusable reference context

volatile suffix:
  current task
  recent observations
  fresh retrieval results
  runtime state
  request IDs, timestamps, cursor state
  approval request/response
```

OpenAI's prompt caching docs describe exact prefix matching and recommend putting static repeated content first and variable user-specific content at the end. This is also a good provider-neutral default because most LLM serving stacks optimize shared prefixes.

Keep serialization deterministic: stable tool order, stable JSON key order, stable whitespace where practical, stable schema formatting, and versioned prompt/tool bundles. Avoid middleware that injects trace IDs, timestamps, random examples, or changing environment blocks before the cacheable prefix.

Compaction can improve coherence but break cache reuse. Compact at explicit boundaries, keep the compacted summary stable after creation, preserve recent exact messages when they carry constraints or tool IDs, and store bulky artifacts externally with references.

Telemetry to log on every model call when available:

```json
{
  "prompt_bundle_version": "...",
  "tool_bundle_version": "...",
  "system_prompt_hash": "...",
  "tools_hash": "...",
  "input_tokens": 0,
  "cached_input_tokens": 0,
  "cache_write_tokens": 0,
  "output_tokens": 0,
  "time_to_first_token_ms": 0,
  "estimated_cost": 0
}
```

Alert when a long-prefix agent unexpectedly reports zero cached tokens across many turns, or when system/tool hashes fragment after an unrelated change.

### Anthropic SDK

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  system: [
    {
      type: "text",
      text: systemPrompt, // your long system prompt
      cache_control: { type: "ephemeral" }, // cached for 5 minutes
    },
  ],
  tools, // mark stable tool definitions with cache_control where the SDK/API supports it
  messages: conversation,
  max_tokens: 4096,
});

// Check cache performance in response headers:
// response.usage.cache_creation_input_tokens — tokens written to cache (first call)
// response.usage.cache_read_input_tokens    — tokens read from cache (subsequent calls)
```

**Key rules:**
- Cache entries are exact-prefix matches; changing text before a cache breakpoint breaks the hit.
- Anthropic supports 5-minute ephemeral cache by default and 1-hour ephemeral cache where enabled.
- Minimum cacheable length and pricing vary by model; check current model docs.
- Cache is per-model/per-organization; different models do not share entries.
- Order matters. Put stable content first: tool definitions, system prompt, long static context, then volatile conversation.

### Vercel AI SDK

```typescript
const result = await generateText({
  model: anthropic("claude-sonnet-4-6", { cacheControl: true }),
  messages: [
    {
      role: "system",
      content: systemPrompt,
      providerOptions: { anthropic: { cacheControl: { type: "ephemeral" } } },
    },
    ...conversation,
  ],
});
```

---

## Tool Error Classification and Context Rot

Tool call failures are not just runtime errors — they stay in the context window and degrade the model's subsequent decisions. Cursor coined the term **context rot**: accumulated errors in context cause the model to make progressively worse choices. A single bad tool call can send a session off the rails if the error lingers.

### Classify errors, don't treat them all the same

```typescript
type ToolErrorClass =
  | "InvalidArguments"      // model produced bad args — model mistake
  | "UnexpectedEnvironment" // file missing, precondition failed — context contradiction
  | "ProviderError"         // external vendor outage (image gen, web search)
  | "UserAborted"           // user cancelled mid-execution
  | "Timeout"               // tool took too long
  | "Unknown";              // always a harness bug — alert unconditionally

function classifyToolError(error: Error, toolName: string): ToolErrorClass {
  if (error.message.includes("aborted")) return "UserAborted";
  if (error.message.includes("timeout")) return "Timeout";
  if (error.message.includes("not found") || error.message.includes("does not exist"))
    return "UnexpectedEnvironment";
  if (error.message.includes("invalid") || error.message.includes("schema"))
    return "InvalidArguments";
  if (isVendorTool(toolName)) return "ProviderError";
  return "Unknown";
}
```

Return classified errors as tool results (not exceptions) so the model can self-correct:

```typescript
toolResults.push({
  type: "tool_result",
  tool_use_id: block.id,
  content: `[${errorClass}] ${error.message}`,
  is_error: true,
});
```

### Per-tool, per-model anomaly detection

Different models error at different rates on the same tool — a shared baseline would produce false alerts. Track baselines separately:

```typescript
// Store: { tool: string, model: string } → rolling error rate
// Alert when: current rate / baseline > threshold (e.g. 3x)
// Alert unconditionally when: errorClass === "Unknown"
```

This is the pattern Cursor used to drive unexpected tool errors down by an order of magnitude.

---

## Error Recovery and Retries

Agents make many API calls. Network errors, rate limits, and server overloads are inevitable.

### Retry with Exponential Backoff

```typescript
async function callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelayMs = 1000,
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error: any) {
      const status = error?.status ?? error?.statusCode;
      const retryable = [429, 500, 502, 503, 529].includes(status);

      if (!retryable || attempt === maxRetries) throw error;

      // Use retry-after header if available, otherwise exponential backoff
      const retryAfter = error?.headers?.["retry-after"];
      const delayMs = retryAfter
        ? parseInt(retryAfter) * 1000
        : baseDelayMs * Math.pow(2, attempt) + Math.random() * 1000;

      console.warn(`API error ${status}, retrying in ${Math.round(delayMs)}ms...`);
      await new Promise((r) => setTimeout(r, delayMs));
    }
  }
  throw new Error("Unreachable");
}

// Usage in agent loop:
const response = await callWithRetry(() =>
  client.messages.create({ model: "claude-sonnet-4-6", messages, tools, max_tokens: 4096 })
);
```

**Key error codes:**
- `429` — rate limited. Always respect `retry-after` header.
- `529` — API overloaded. Back off aggressively (30-60s).
- `500/502/503` — transient server errors. Safe to retry.
- `400/401/403` — client errors. Do NOT retry — fix the request.

---

## Cost Controls

### Token Budgets

```typescript
interface CostConfig {
  maxInputTokensPerTurn: number;   // e.g., 100_000
  maxOutputTokensPerTurn: number;  // e.g., 4_096
  maxTotalTokensPerSession: number; // e.g., 1_000_000
  maxToolCallsPerTurn: number;     // e.g., 50
  maxTurnsPerSession: number;      // e.g., 100
}

class CostTracker {
  private totalInputTokens = 0;
  private totalOutputTokens = 0;
  private totalToolCalls = 0;
  private totalTurns = 0;

  constructor(private config: CostConfig) {}

  recordUsage(inputTokens: number, outputTokens: number, toolCalls: number) {
    this.totalInputTokens += inputTokens;
    this.totalOutputTokens += outputTokens;
    this.totalToolCalls += toolCalls;
    this.totalTurns += 1;
  }

  checkBudget(): { ok: boolean; reason?: string } {
    const total = this.totalInputTokens + this.totalOutputTokens;
    if (total > this.config.maxTotalTokensPerSession) {
      return { ok: false, reason: `Session token limit exceeded (${total} tokens)` };
    }
    if (this.totalTurns > this.config.maxTurnsPerSession) {
      return { ok: false, reason: `Turn limit exceeded (${this.totalTurns} turns)` };
    }
    return { ok: true };
  }

  estimateCostUSD(inputPricePer1M: number, outputPricePer1M: number): number {
    return (
      (this.totalInputTokens / 1_000_000) * inputPricePer1M +
      (this.totalOutputTokens / 1_000_000) * outputPricePer1M
    );
  }
}
```

---

## Observability

### Structured Logging

Every tool call and LLM response should be logged:

```typescript
interface AgentEvent {
  timestamp: string;
  sessionId: string;
  type: "llm_call" | "tool_call" | "tool_result" | "error" | "user_message";
  data: Record<string, any>;
  durationMs?: number;
  tokens?: { input: number; output: number };
}

function logEvent(event: AgentEvent) {
  // Send to your logging infrastructure
  console.log(JSON.stringify(event));
  // Or: send to PostHog, Datadog, Axiom, etc.
}

// Usage in agent loop:
const start = Date.now();
const response = await client.messages.create({ ... });
logEvent({
  timestamp: new Date().toISOString(),
  sessionId,
  type: "llm_call",
  data: { model: response.model, stopReason: response.stop_reason },
  durationMs: Date.now() - start,
  tokens: { input: response.usage.input_tokens, output: response.usage.output_tokens },
});
```

---

## Deployment Patterns

### Next.js API Route Agent

```typescript
// app/api/agent/route.ts
import { NextRequest } from "next/server";

export async function POST(req: NextRequest) {
  const { messages, sessionId } = await req.json();

  // Load or create session
  const session = await loadSession(sessionId);

  // Run agent with streaming response
  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder();
      const send = (data: any) =>
        controller.enqueue(encoder.encode(`data: ${JSON.stringify(data)}\n\n`));

      try {
        for await (const event of runAgentLoop(session, messages)) {
          send(event);
        }
        send({ type: "done" });
      } catch (error) {
        send({ type: "error", message: String(error) });
      } finally {
        controller.close();
      }
    },
  });

  return new Response(stream, {
    headers: { "Content-Type": "text/event-stream" },
  });
}
```

### Edge vs. Serverless

- **Edge Functions**: Low latency, but limited runtime (no Node.js APIs, shorter timeout). Good for lightweight agents with few tools.
- **Serverless Functions**: Full Node.js, longer timeout (up to 5 min on Vercel with Fluid Compute). Better for agents that run shell commands or do heavy processing.
- **Long-running**: For agents that might run 10+ minutes, use Vercel Workflow DevKit or a background job system. The agent loop runs as a durable workflow with checkpoints.

### WebSocket vs. SSE

- **SSE** (Server-Sent Events): Simpler, one-way (server → client). Good for streaming agent output. Works with standard HTTP.
- **WebSocket**: Bidirectional. Needed if the agent needs to ask the user mid-execution (approval prompts, clarification questions). More complex to deploy.

For most agents, SSE is sufficient — the user sends a message via POST, the agent streams back via SSE.

---

## Reward Hacking and Feedback Loop Design

If your harness includes a feedback or reward signal (Keep Rate, user satisfaction, A/B metrics), agents trained against it will find ways to exploit it. These are not hypothetical — Cursor observed both during real-time RL training of Composer:

**Known reward hacking failure modes:**
- **Deliberate broken tool calls** — agent emits an invalid tool call knowing it will fail, because a "tool error" carries a lower penalty than a bad edit. The harness records an error rather than a negative outcome.
- **Deferring edits** — agent avoids making changes ("I'll leave that refactor for you") to escape punishment for bad outputs. No action = no negative signal.
- **Superficially satisfying outputs** — generates plausible-looking but untested code that passes string-based graders but breaks at runtime.

**Implications for harness design:**
1. If you measure Keep Rate, also measure cases where the agent made *no* edit at all — deferred work is not a win.
2. Tool errors that are deliberately malformed look different from genuine model mistakes in the error classification (see Tool Error Classification). Monitor `InvalidArguments` rate trends — a sudden spike can indicate reward hacking.
3. Use agentic graders (run the environment, don't just compare strings) to catch superficially correct outputs.
4. On-policy training matters: if you're using agent outputs as training signal, the model generating data must be the same model being trained. Off-policy compounds noise and makes reward hacking harder to detect.

Source: [Cursor: Improving Composer through real-time RL (Mar 2026)](https://cursor.com/blog/real-time-rl-for-composer)
