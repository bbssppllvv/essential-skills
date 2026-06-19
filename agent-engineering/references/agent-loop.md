# Agent Loop — Deep Dive

Evidence basis:
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic tool use docs](https://platform.claude.com/docs/en/docs/agents-and-tools/tool-use/overview)
- [Vercel AI SDK Agents docs](https://ai-sdk.dev/docs/agents/overview)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [OpenAI: Unrolling the Codex agent loop](https://openai.com/index/unrolling-the-codex-agent-loop/)

API snippets are examples. Verify exact model IDs, imports, and helper names against official docs before shipping.

## Table of Contents
1. [Loop Variants](#loop-variants)
2. [Conversation Management](#conversation-management)
3. [Turn Budget and Limits](#turn-budget-and-limits)
4. [Thinking and Planning](#thinking-and-planning)

---

## Loop Variants

### Basic ReAct (Reason + Act)

The standard agent loop from the main skill doc. The LLM alternates between reasoning (text) and acting (tool calls). Most production agents use this pattern.

```
Think → Act → Observe → Think → Act → Observe → ... → Respond
```

The LLM's visible text between tool calls is narration or a plan, not guaranteed complete reasoning. It is useful for progress visibility and debugging, but the harness should treat tool results, structured state, and eval signals as the source of truth.

### Plan-then-Execute

A two-phase approach:
1. **Planning phase**: LLM receives the task, produces a step-by-step plan (text only, no tool calls)
2. **Execution phase**: LLM follows its own plan, calling tools for each step

```typescript
// Phase 1: Plan
conversation.push({ role: "user", content: `${task}\n\nFirst, create a detailed plan.` });
const planResponse = await callLLM(conversation, []); // No tools available during planning
const plan = planResponse.text;

// Phase 2: Execute
conversation.push({ role: "user", content: `Now execute this plan:\n${plan}` });
while (true) { // Standard inner loop with tools enabled
  const response = await callLLM(conversation, allTools);
  // ...
}
```

**When to use**: Complex multi-file refactors, tasks where wrong first steps are expensive, when you want the user to approve a plan before execution.

**Tradeoff**: More expensive (extra LLM call for planning), but reduces wasted work on wrong approaches.

### Iterative Refinement

The agent produces output, evaluates it, and improves it in a loop:

```
Generate → Evaluate → Improve → Evaluate → ... → Good enough → Done
```

Useful for creative tasks, code that needs to pass tests, or output that has a measurable quality signal.

```typescript
let attempts = 0;
while (attempts < maxAttempts) {
  // Generate/improve
  const response = await callLLM(conversation, tools);
  await executeToolCalls(response);

  // Evaluate
  const result = await runTests(); // or any quality check
  if (result.passed) break;

  // Feed failure back
  conversation.push({
    role: "user",
    content: `Tests failed:\n${result.errors}\nPlease fix.`,
  });
  attempts++;
}
```

### Human-in-the-Loop

The agent pauses at decision points and asks the user:

```typescript
while (true) {
  const response = await callLLM(conversation, tools);
  const toolCalls = extractToolCalls(response);

  if (!toolCalls.length) {
    console.log(response.text);
    break;
  }

  for (const tc of toolCalls) {
    if (needsApproval(tc)) {
      const approved = await askUser(`Execute ${tc.name}(${JSON.stringify(tc.input)})?`);
      if (!approved) {
        conversation.push(toolResult(tc.id, "User denied this action."));
        continue;
      }
    }
    const result = executeTool(tc.name, tc.input);
    conversation.push(toolResult(tc.id, result));
  }
}
```

This is how most production coding agents work — reads are auto-approved, writes need user confirmation.

---

## Conversation Management

### The Conversation as Memory

The model's request context is the agent's working memory. Everything the model can use at inference time comes from:
1. The system prompt (persistent instructions)
2. The conversation history (what happened so far)
3. Tool results (what it observed in the world)

The model has no private persistent state across calls. The harness may have durable state, but anything the model needs for the next decision must be included directly, referenced through retrievable IDs, or available through a tool.

### Message Roles

Message formats are API-specific. The conceptual roles are:
- **system**: Persistent instructions, personality, rules. Set once at the start.
- **user**: Human messages. In Anthropic's Messages API, tool results are also `tool_result` blocks inside a user message.
- **tool/result items**: Other APIs and frameworks may use a dedicated `tool` role or response item type. Do not blindly port Anthropic message structure to OpenAI, AI SDK, or LangChain.
- **assistant**: LLM responses — both text and tool calls.

The conversation alternates: user → assistant → user → assistant. Tool call flows look like:

```
user: "Fix the bug in auth.py"
assistant: [tool_use: read_file("auth.py")]
user: [tool_result: "contents of auth.py..."]
assistant: [text: "I see the issue"] [tool_use: edit_file(...)]
user: [tool_result: "edit applied successfully"]
assistant: [text: "Fixed. The bug was..."]
```

### Context Window Budget

A typical budget allocation:
- **System prompt**: 2K-8K tokens (instructions, tool definitions)
- **Recent conversation**: 60-80% of remaining window
- **Tool results**: Can be huge — truncate aggressively
- **Reserve**: Always leave ~4K tokens for the LLM's response

### Compaction Strategies

When the conversation approaches the context limit:

**Strategy 1: Sliding Window**
Drop the oldest messages, keep the last N turns. Simple but loses early context.

```typescript
if (tokenCount(conversation) > MAX_TOKENS * 0.8) {
  conversation = conversation.slice(-N); // keep last N messages
}
```

**Strategy 2: Summarize-and-Drop**
Ask the LLM to summarize the conversation so far, replace old messages with the summary.

```typescript
if (tokenCount(conversation) > MAX_TOKENS * 0.8) {
  const summary = await callLLM([
    { role: "user", content: `Summarize this conversation:\n${formatMessages(conversation.slice(0, cutoff))}` },
  ]);
  conversation = [
    { role: "user", content: `Previous context summary:\n${summary}` },
    { role: "assistant", content: "Understood." },
    ...conversation.slice(cutoff),
  ];
}
```

**Strategy 3: Tool Result Truncation**
Old tool results are rarely needed verbatim. Replace them with summaries.

```typescript
for (const msg of conversation.slice(0, -RECENT_WINDOW)) {
  if (isToolResult(msg)) {
    msg.content = truncate(msg.content as string, 200);
  }
}
```

**In practice**, combine all three: truncate old tool results first, then summarize if still too long, then drop as a last resort.

---

## Turn Budget and Limits

### Why Limits Matter

An agent without limits can:
- Spin in infinite loops calling the same failing tool
- Rack up massive API costs on a single query
- Take 20 minutes on something the user expected to take 20 seconds

### What to Limit

- **Max inner-loop iterations** (agent turns per user message): 25-50 is typical
- **Max total tokens per session**: Depends on your budget, but set something
- **Max tool execution time**: 30-120 seconds per tool call
- **Max total session time**: 10-30 minutes for coding agents

### Stuck Loop Detection

```typescript
const recentCalls: string[] = [];

while (true) {
  const response = await callLLM(conversation, tools);
  const toolCalls = extractToolCalls(response);

  if (toolCalls.length) {
    const signature = JSON.stringify(
      toolCalls.map((tc) => [tc.name, JSON.stringify(tc.input)])
    );
    recentCalls.push(signature);

    // Check for repetition
    if (
      recentCalls.length >= 3 &&
      recentCalls.at(-1) === recentCalls.at(-2) &&
      recentCalls.at(-2) === recentCalls.at(-3)
    ) {
      conversation.push({
        role: "user",
        content:
          "You appear to be stuck in a loop, repeating the same tool calls. " +
          "Please try a different approach or explain what's blocking you.",
      });
      recentCalls.length = 0;
    }
  }
  // ... rest of loop
}
```

---

## Thinking and Planning

### Extended Thinking

Some models support "thinking" — internal reasoning that's separate from the visible output. This is powerful for agents because it lets the model plan without cluttering the conversation.

With Claude's extended thinking:
```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  messages: conversation,
  tools,
  thinking: {
    type: "enabled",
    budget_tokens: 10000, // tokens for internal reasoning
  },
});
// response.content may contain thinking blocks (internal) + text/tool_use blocks (external)
```

**When to use**: Complex multi-step tasks, tasks requiring careful planning, tasks where mistakes are expensive.

**When NOT to use**: Simple tasks (wasted tokens), high-throughput scenarios (added latency).

### Scratchpad Pattern

If extended thinking isn't available, you can add a private note/planning tool, but be precise about privacy: the content is still stored in transcripts/logs unless your harness redacts it. Use it for concise plans and state, not sensitive chain-of-thought promises.

```typescript
const thinkTool = {
  name: "think",
  description:
    "Record a concise private plan or decision note for this task. " +
    "Use it when a plan, hypothesis, or trade-off needs to persist " +
    "without cluttering the user-facing response.",
  input_schema: {
    type: "object" as const,
    properties: {
      note: { type: "string", description: "Concise plan, hypothesis, or decision note" },
    },
    required: ["note"],
  },
};
```

The tool can return "OK" — the value is creating a compact state artifact the model can reference later. Keep these notes short and scrub them from user-visible logs unless you intentionally expose them.
