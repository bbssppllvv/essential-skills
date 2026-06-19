# Anthropic's Official Patterns — Reference

Canonical source: [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)

## Table of Contents
1. [The 6 Building Blocks + Autonomous Agents](#the-6-building-blocks--autonomous-agents)
2. [Progressive Tool Use Guide](#progressive-tool-use-guide)
3. [Claude Agent SDK](#claude-agent-sdk)
4. [Long-Running Harness Patterns](#long-running-harness-patterns)
5. [Context Engineering Strategies](#context-engineering-strategies)
6. [Multi-Agent System Design](#multi-agent-system-design)
7. [Evaluation Methodology](#evaluation-methodology)

---

## The 6 Building Blocks + Autonomous Agents

### 1. Augmented LLM (Foundation)
An LLM combined with retrieval, tools, and memory. The basic unit everything else builds on.

### 2. Prompt Chaining (Workflow)
Sequential LLM calls where each processes the prior output. Optional programmatic gates between steps. **When**: tasks that decompose into fixed subtasks. Trades latency for accuracy.

### 3. Routing (Workflow)
Classify input, direct to specialized handlers. **Purpose**: separation of concerns — optimizing for one input type doesn't hurt another. Example: easy questions → Haiku, hard → Opus.

### 4. Parallelization (Workflow)
Two forms:
- **Sectioning**: independent subtasks run simultaneously
- **Voting**: same task run N times for diverse outputs

Key insight: "For complex tasks with multiple considerations, LLMs generally perform better when each consideration is handled by a separate LLM call."

### 5. Orchestrator-Workers (Workflow)
Central LLM dynamically breaks tasks and delegates to workers. Unlike parallelization, subtasks aren't pre-defined but determined by the orchestrator based on input.

### 6. Evaluator-Optimizer (Workflow)
One LLM generates, another evaluates and provides feedback in a loop. Only use when: clear evaluation criteria exist AND responses demonstrably improve with feedback.

### Autonomous Agents
The full agent pattern. "Just LLMs using tools based on environmental feedback in a loop." Use for open-ended problems where step count is unpredictable. Higher costs, potential for compounding errors.

---

## Progressive Tool Use Guide

A progressive tutorial for building a tool-using agent — from a single tool call to a full production loop.

### Ring 1: Single tool, single turn
One tool, one message, one tool call. Learn the API shape.

### Ring 2: The Agentic Loop
The core `while (response.stop_reason === "tool_use")` pattern. The model chains tool calls until it decides to respond with text.

### Ring 3: Multiple tools, parallel calls
When Claude returns multiple `tool_use` blocks, process ALL and send ALL results in one message:

```typescript
while (response.stop_reason === "tool_use") {
  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const block of response.content) {
    if (block.type === "tool_use") {
      const result = executeTool(block.name, block.input);
      toolResults.push({ type: "tool_result", tool_use_id: block.id, content: JSON.stringify(result) });
    }
  }
  messages.push({ role: "assistant", content: response.content });
  messages.push({ role: "user", content: toolResults });
  response = await client.messages.create({ ...params, messages });
}
```

### Ring 4: Error handling
Send errors with `is_error: true`. Claude reads the error and can retry, adjust, or explain:

```typescript
try {
  const result = executeTool(block.name, block.input);
  toolResults.push({ type: "tool_result", tool_use_id: block.id, content: JSON.stringify(result) });
} catch (err) {
  toolResults.push({ type: "tool_result", tool_use_id: block.id, content: String(err), is_error: true });
}
```

### Ring 5: Tool Runner SDK
The SDK's `toolRunner` handles the entire loop:

```typescript
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const myTool = betaZodTool({
  name: "create_event",
  description: "Create a calendar event",
  inputSchema: z.object({ title: z.string(), start: z.string().datetime() }),
  run: async (input) => JSON.stringify({ event_id: "evt_123", title: input.title }),
});

const finalMessage = await client.beta.messages.toolRunner({
  model: "claude-opus-4-7",
  max_tokens: 1024,
  tools: [myTool],
  messages: [{ role: "user", content: "Schedule a planning session" }],
});
```

---

## Claude Agent SDK

The same agent loop that powers Claude Code, programmable as a library.

### TypeScript
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Refactor the auth module",
  options: {
    allowedTools: ["Read", "Edit", "Bash", "Glob", "Grep"],
    permissionMode: "acceptEdits",
  },
})) {
  console.log(message);
}
```

### Key Features

**Hooks** — lifecycle callbacks:
```typescript
options: {
  hooks: {
    PostToolUse: [{ matcher: "Edit|Write", hooks: [auditLogger] }],
  },
}
```

**Subagents** — isolated context for subtasks:
```typescript
options: {
  allowedTools: ["Read", "Glob", "Agent"],
  agents: {
    "code-reviewer": {
      description: "Reviews code for quality and security",
      prompt: "Analyze code quality and suggest improvements.",
      tools: ["Read", "Glob", "Grep"],
    },
  },
}
```

**Sessions** — resume with full context:
```typescript
// Resume a previous session
for await (const message of query({
  prompt: "Now find all places that call it",
  options: { resume: sessionId },
})) { ... }
```

**Custom MCP tools** — add tools without subprocess overhead:
```typescript
import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";

const myServer = createSdkMcpServer({
  name: "my-tools",
  tools: [greetTool, lookupTool],
});
```

---

## Long-Running Harness Patterns

Source: [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)

### Two-Phase Architecture
1. **Initializer Agent**: Sets up environment, creates feature list, writes init.sh, makes initial commit
2. **Coding Agent**: Makes incremental progress per session, reads progress files and feature list

### Critical Finding
"Compaction isn't sufficient. Even a frontier model running in a loop across multiple context windows will fall short of building a production-quality web app if only given a high-level prompt."

### Session Startup Sequence
Every session: `pwd` → read git logs & progress files → identify next feature → run init.sh → verify environment.

### State Bridging Between Sessions
- Descriptive git commits capturing incremental changes
- Progress file summaries documenting completed work
- **JSON over Markdown** for structured state (lower corruption rates)
- **Single feature per session** — prevents context exhaustion from one-shotting everything

### Verification Is The Highest-Leverage Practice
"The single highest-leverage thing you can do." Agents perform dramatically better when they can verify their own work — run tests, compare screenshots, validate outputs.

---

## Context Engineering Strategies

Source: [Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

### "Right Altitude" Principle
Two failure modes:
- **Brittle**: hardcoded complex logic → fragility
- **Vague**: overly general guidance → insufficient signals

Optimal: specific enough to guide, flexible enough to provide strong heuristics.

### Just-in-Time Context
Agents maintain lightweight identifiers (file paths, stored queries) and load dynamically via tools. Claude Code analyzes data by writing targeted queries and using `head`/`tail`, avoiding loading full objects.

### Structured Note-Taking (Agentic Memory)
Agent writes notes to external storage, retrieves when relevant. Enables persistent memory with minimal overhead. Example: Claude playing Pokemon tracked "for the last 1,234 steps I've been training my Pokemon in Route 1, Pikachu gained 8 levels toward the target of 10."

### Sub-Agent Context Isolation
Main agent coordinates with high-level plan. Sub-agents do deep work in separate context windows. Each may use tens of thousands of tokens but returns only a 1,000-2,000 token summary. The lead synthesizes.

### Compaction Tuning
Start by maximizing recall, iterate to improve precision. Tool result clearing is "one of the safest lightest touch forms of compaction."

---

## Multi-Agent System Design

Source: [How We Built Our Multi-Agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system)

### Architecture
- Lead: Claude Opus (orchestration + synthesis)
- Sub-agents: Claude Sonnet (specialized exploration)
- 3-5 sub-agents spawned in parallel
- Each has separate context window
- Returns condensed findings to lead

### Performance
Multi-agent **outperformed single-agent by 90.2%**. Token usage explains 80% of variance. Tool calls + model choice explain the remaining 20%.

### 8 Key Practices
1. **Think like your agents** — build simulations, watch step-by-step
2. **Detailed delegation** — provide objective, output format, tool guidance, boundaries
3. **Scale effort to query complexity** — embed scaling rules in prompts
4. **Tool design is critical** — bad descriptions send agents "completely wrong paths"
5. **Let agents improve themselves** — Claude 4+ models are excellent prompt engineers
6. **Start wide, then narrow** — mirror expert research patterns
7. **Guide thinking** — extended thinking as controllable scratchpad
8. **Parallel tool calling** — 3+ parallel tools per sub-agent, up to 90% time reduction

### Sub-Agent Output Pattern
"Subagent output to a filesystem to minimize the 'game of telephone'" — agents create persistent outputs rather than passing through the lead agent.

---

## Evaluation Methodology

Source: [Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

### Eight-Step Roadmap
1. Start early with 20-50 tasks from real failures
2. Convert manual tests from development and bug reports
3. Write unambiguous tasks (domain experts independently reach same verdict)
4. Build balanced problem sets (positive AND negative cases)
5. Create robust harnesses with isolated trials from clean environments
6. Design graders thoughtfully — prefer deterministic, calibrate LLM judges against humans
7. Read transcripts regularly to verify graders
8. Monitor for saturation — when pass rates hit 100%, graduate to regression suites

### Metrics
- **pass@k**: probability of 1+ correct in k attempts (development metric)
- **pass^k**: probability ALL k succeed (production reliability metric)
- **0% pass@100** usually means broken task, not broken agent
- **Partial credit matters** — identifying the problem but failing the fix is meaningfully better than immediate failure

### Code Execution via MCP: Token Efficiency
For large tool surfaces, prefer discovery over eager schema loading: expose a small search/list interface first, then load only the selected tool schemas or API docs. This preserves context and makes tool descriptions easier to keep current.

---

## Three Core Design Principles

1. **Simplicity** — maintain simplicity in design
2. **Transparency** — explicitly show planning steps
3. **Documentation Quality** — invest in ACI as much as HCI

> "Success isn't about building the most sophisticated system. It's about building the right system for your needs."
