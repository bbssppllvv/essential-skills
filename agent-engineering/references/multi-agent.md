# Multi-Agent Architectures — Deep Dive

When a single agent loop isn't enough: patterns for coordinating multiple LLM agents.

Evidence basis:
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic: Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [OpenAI Agents SDK handoffs](https://openai.github.io/openai-agents-js/guides/handoffs/)
- [Vercel AI SDK Subagents](https://ai-sdk.dev/docs/agents/subagents)

Multi-agent advice is workload-dependent. Treat reported token and performance numbers as source-scoped evidence, then validate against your eval suite.

## Table of Contents
1. [When to Go Multi-Agent](#when-to-go-multi-agent)
2. [Orchestrator + Workers](#orchestrator--workers)
3. [Pipeline Architecture](#pipeline-architecture)
4. [Peer Swarm](#peer-swarm)
5. [Delegation Patterns](#delegation-patterns)
6. [Coordination and State](#coordination-and-state)
7. [Cost and Latency](#cost-and-latency)

---

## When to Go Multi-Agent

### Signs You Need Multiple Agents

- **Context pollution**: One subtask's tool results are crowding out context needed for another subtask
- **Parallelism opportunity**: Two independent research tasks could run simultaneously
- **Specialized knowledge**: Different parts of the task need different system prompts or tool sets
- **Isolation requirement**: You want a sub-task to fail without corrupting the main conversation
- **Scale**: The task involves processing many items independently (e.g., reviewing 20 files)

### Signs You DON'T Need Multiple Agents

- The task is sequential (each step depends on the previous)
- A single agent with good tools can handle it in one conversation
- You're adding multi-agent just because it sounds impressive (the most common mistake)

**Rule of thumb**: Start with a single agent. Only add agents when you hit a specific limit that multi-agent solves.

---

## Orchestrator + Workers

The most common multi-agent pattern. A primary agent (orchestrator) delegates subtasks to specialized sub-agents (workers).

```
Orchestrator
├── Worker A (research)
├── Worker B (code review)
└── Worker C (testing)
```

### Implementation

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

// The "delegate" tool available to the orchestrator
const delegateTool = {
  name: "delegate",
  description:
    "Spawn a sub-agent to handle a specific subtask. The sub-agent gets its own conversation " +
    "and tools. It does NOT have access to the main conversation — brief it fully in the prompt. " +
    "Returns the sub-agent's final text response.",
  input_schema: {
    type: "object" as const,
    properties: {
      task: {
        type: "string",
        description: "Complete description of what the sub-agent should do. Include all context it needs.",
      },
      tools: {
        type: "array",
        items: { type: "string" },
        description: "Which tools the sub-agent should have access to",
      },
    },
    required: ["task", "tools"],
  },
};

async function runSubAgent(task: string, toolNames: string[]): Promise<string> {
  const subConversation: Anthropic.MessageParam[] = [
    { role: "user", content: task },
  ];
  const subTools = allTools.filter((t) => toolNames.includes(t.name));

  // Run the sub-agent's own inner loop
  while (true) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-6",
      system: "You are a specialized sub-agent. Complete the task and return a concise summary.",
      messages: subConversation,
      tools: subTools,
      max_tokens: 4096,
    });

    subConversation.push({ role: "assistant", content: response.content });

    const toolCalls = response.content.filter(
      (b): b is Anthropic.ToolUseBlock => b.type === "tool_use"
    );

    if (!toolCalls.length) {
      // Sub-agent is done — extract text response
      const text = response.content
        .filter((b): b is Anthropic.TextBlock => b.type === "text")
        .map((b) => b.text)
        .join("\n");
      return text;
    }

    // Execute sub-agent's tool calls
    const results: Anthropic.ToolResultBlockParam[] = toolCalls.map((tc) => ({
      type: "tool_result" as const,
      tool_use_id: tc.id,
      content: String(executeTool(tc.name, tc.input as Record<string, any>)),
    }));
    subConversation.push({ role: "user", content: results });
  }
}
```

### Key Design Decisions

**What the orchestrator sees**: Only the sub-agent's final text response. It doesn't see the sub-agent's tool calls, intermediate reasoning, or errors that were recovered from. This keeps the orchestrator's context clean.

**Keep the coordination protocol minimal**: Cursor's multi-agent GPU optimization system (38% speedup across 235 problems) specified the entire inter-agent protocol in a single markdown file — just output format, rules, and tests. Workers autonomously learned to call the benchmarking pipeline themselves, creating a self-testing loop without developer intervention. Start with a minimal protocol and only add structure when agents demonstrate they need it. Over-engineering coordination upfront is a common mistake.

**Dynamic rebalancing over static assignment**: Don't split work equally upfront. The planner in Cursor's system redistributed tasks based on performance metrics as workers completed them. Design the orchestrator to rebalance load dynamically.

Source: [Cursor: Speeding up GPU kernels by 38% with a multi-agent system (Apr 2026)](https://cursor.com/blog/multi-agent-kernels)

**Briefing the sub-agent**: The sub-agent starts with zero context. Everything it needs must be in the task description:
```typescript
// BAD: "Fix the auth bug" — sub-agent doesn't know which bug or where
// GOOD: "In /src/auth/login.ts, the validateToken function on line 42 throws 
//        TypeError when token is undefined. Read the file, fix the null check, 
//        and verify the fix handles both undefined and empty string cases."
```

---

## Pipeline Architecture

Agents in sequence, each transforming the output of the previous one. Good for workflows with distinct phases.

```
Planner → Coder → Reviewer → Fixer → Done
```

### Implementation

```typescript
interface PipelineStage {
  name: string;
  systemPrompt: string;
  tools: string[];
  transform: (input: string) => string; // how to phrase the task for this stage
}

const pipeline: PipelineStage[] = [
  {
    name: "planner",
    systemPrompt: "You are a software architect. Create detailed implementation plans.",
    tools: ["read_file", "glob", "grep"],
    transform: (task) => `Create a step-by-step implementation plan for: ${task}`,
  },
  {
    name: "coder",
    systemPrompt: "You are a senior developer. Implement code changes precisely.",
    tools: ["read_file", "edit_file", "write_file", "bash"],
    transform: (plan) => `Implement this plan:\n\n${plan}`,
  },
  {
    name: "reviewer",
    systemPrompt: "You are a code reviewer. Find bugs, security issues, and style problems.",
    tools: ["read_file", "grep", "bash"],
    transform: (summary) =>
      `Review the changes just made:\n\n${summary}\n\nList any issues found.`,
  },
];

async function runPipeline(initialTask: string): Promise<string> {
  let input = initialTask;

  for (const stage of pipeline) {
    console.log(`\n--- Stage: ${stage.name} ---`);
    const prompt = stage.transform(input);
    input = await runSubAgent(prompt, stage.tools); // reuse sub-agent runner
    console.log(`${stage.name} output: ${input.slice(0, 200)}...`);
  }

  return input;
}
```

### When to Use

- Code generation workflows (plan → implement → test → review)
- Document processing (extract → transform → validate → output)
- Data analysis (gather → clean → analyze → report)

### Pitfall: Lost Context

Each stage only sees the previous stage's output, not the original task. If Stage 3 needs info from Stage 1 that Stage 2 didn't pass through, it's lost. Mitigate by:
- Including the original task at every stage
- Having each stage produce structured output that carries forward

---

## Peer Swarm

Multiple agents working independently on the same problem, coordinating through shared state (usually the filesystem).

```
Agent A ──┐
Agent B ──┼──→ Shared Filesystem ──→ Merge
Agent C ──┘
```

### Implementation

```typescript
interface SwarmTask {
  id: string;
  description: string;
  files: string[]; // files this agent is responsible for
}

async function runSwarm(tasks: SwarmTask[]): Promise<void> {
  // Launch all agents in parallel
  const results = await Promise.allSettled(
    tasks.map(async (task) => {
      const prompt = [
        `You are responsible for: ${task.description}`,
        `You may ONLY edit these files: ${task.files.join(", ")}`,
        `Other agents are working on other files simultaneously.`,
        `Do not modify files outside your scope.`,
      ].join("\n");

      return runSubAgent(prompt, ["read_file", "edit_file", "glob", "grep"]);
    })
  );

  // Check for conflicts
  for (const result of results) {
    if (result.status === "rejected") {
      console.error("Agent failed:", result.reason);
    }
  }
}

// Example: parallel file review
const reviewTasks: SwarmTask[] = [
  { id: "auth", description: "Review auth module for security issues", files: ["src/auth/**"] },
  { id: "api", description: "Review API routes for performance issues", files: ["src/api/**"] },
  { id: "db", description: "Review database queries for N+1 problems", files: ["src/db/**"] },
];

await runSwarm(reviewTasks);
```

### Conflict Prevention

The biggest risk with peer swarms: two agents editing the same file simultaneously.

**Strategy 1: File ownership** — Each agent has exclusive write access to a set of files. Simple, prevents all conflicts.

**Strategy 2: Merge-on-complete** — Each agent works on a git branch. After all agents finish, merge branches. Handle conflicts like you would with human developers.

**Strategy 3: Shared lock** — A central mutex prevents concurrent writes to the same file. More complex but more flexible.

---

## Delegation Patterns

### Fan-Out / Fan-In

Orchestrator breaks a task into N independent subtasks, runs them in parallel, then synthesizes results.

```typescript
async function fanOutFanIn(
  subtasks: string[],
  tools: string[]
): Promise<string> {
  // Fan out
  const results = await Promise.all(
    subtasks.map((task) => runSubAgent(task, tools))
  );

  // Fan in — let the orchestrator synthesize
  const synthesis = await client.messages.create({
    model: "claude-sonnet-4-6",
    messages: [
      {
        role: "user",
        content: `Here are the results from ${results.length} parallel investigations:\n\n${results.map((r, i) => `### Subtask ${i + 1}\n${r}`).join("\n\n")}\n\nSynthesize these into a coherent summary.`,
      },
    ],
    max_tokens: 4096,
  });

  return synthesis.content
    .filter((b): b is Anthropic.TextBlock => b.type === "text")
    .map((b) => b.text)
    .join("\n");
}
```

### Escalation

Sub-agent tries a task. If it can't solve it, escalates to a more capable (and expensive) agent.

```typescript
function subAgentSucceeded(result: string): boolean {
  // Prefer JSON like { "status": "success" | "blocked", "summary": "..." }.
  // This fallback is only for illustration.
  try {
    return JSON.parse(result).status === "success";
  } catch {
    return false;
  }
}

async function withEscalation(
  task: string,
  tools: string[]
): Promise<string> {
  // Try with fast/cheap model first
  const fastResult = await runSubAgent(task, tools, {
    model: "claude-haiku-4-5-20251001",
    maxTurns: 5,
  });

  // In production, require structured status from the sub-agent
  // instead of guessing success from phrases in free text.
  if (subAgentSucceeded(fastResult)) {
    return fastResult;
  }

  // Escalate to powerful model
  console.log("Fast agent couldn't handle it — escalating...");
  return runSubAgent(
    `A previous attempt at this task failed. Here's what was tried:\n${fastResult}\n\nPlease try a different approach:\n${task}`,
    tools,
    { model: "claude-sonnet-4-6", maxTurns: 20 }
  );
}
```

---

## Coordination and State

### Shared State via Filesystem

The simplest coordination mechanism — agents read and write files. This is how most coding agents work.

```
/project/
├── src/          ← agents read and edit code here
├── .agent-state/ ← agents coordinate here
│   ├── plan.md   ← shared plan document
│   ├── done.json ← completed task tracking
│   └── locks/    ← file-level locks
```

### Message Passing

For more complex coordination, agents can communicate through a message queue:

```typescript
// Simple in-memory message bus
const messageQueue = new Map<string, any[]>();

function sendToAgent(agentId: string, message: any) {
  if (!messageQueue.has(agentId)) messageQueue.set(agentId, []);
  messageQueue.get(agentId)!.push(message);
}

function receiveMessages(agentId: string): any[] {
  const messages = messageQueue.get(agentId) || [];
  messageQueue.set(agentId, []);
  return messages;
}

// Exposed as a tool to agents:
const checkMessagesTool = {
  name: "check_messages",
  description: "Check for messages from other agents",
  // ...
};
```

---

## Cost and Latency

### Cost Model

Each sub-agent is a separate LLM conversation. Costs add up fast:

```
Orchestrator:     ~5K input + ~2K output per turn
Sub-agent A:     ~20K input + ~5K output (reads files, iterates)
Sub-agent B:     ~15K input + ~3K output
Sub-agent C:     ~25K input + ~8K output
─────────────────────────────────
Total per cycle:  ~83K tokens

vs. Single agent: ~40K tokens (but sequential, slower)
```

**Rule**: multi-agent cost depends on what you compare. For a tightly scoped task with clean parallel slices, expect roughly several-x overhead versus a single agent. For breadth-first research, Anthropic reported agents using about 4x chat tokens and multi-agent systems about 15x chat tokens. The benefit must come from parallelism, context isolation, or better coverage.

### Latency Optimization

- **Parallel when possible**: `Promise.all()` for independent subtasks
- **Cheap models for simple tasks**: Haiku for formatting, file listing, simple checks
- **Early termination**: If the orchestrator gets what it needs from 2 of 3 agents, cancel the third
- **Streaming aggregation**: Stream sub-agent results to the user as they complete, don't wait for all

### When Single Agent Wins

- Simple tasks (< 10 tool calls)
- Highly sequential tasks (each step depends on previous)
- Cost-sensitive environments
- When context sharing between subtasks is important

### When Multi-Agent Wins

- Large codebase operations (review 50 files)
- Tasks with clear parallel subtasks
- Mixed-skill tasks (research + coding + testing)
- Long-running tasks where one failure shouldn't block everything

---

## Production Lessons (Cognition + Anthropic, 2026)

### Single-threaded writes, distributed intelligence

The core architecture that works in production: multiple agents contribute intelligence (analysis, review, planning) while all writes remain single-threaded through one agent. Parallel writes cause fragmentation — each agent makes implicit decisions about code style, edge cases, and patterns that conflict with siblings.

Source: [Cognition: Multi-Agents: What's Actually Working (Apr 2026)](https://cognition.ai/blog/multi-agents-working)

### Give reviewers LESS context, not more

Counter-intuitive: review agents perform *better* with shorter, cleaner context windows — not the full coder context. Sharing the coder's complete context introduces context rot into the reviewer's window, degrading its attention on the code it's supposed to evaluate. Isolate reviewers to the artifact + criteria, not the full session history.

### Quality ceiling = primary model quality

The orchestrator/primary model sets the ceiling. Weak primary models can't reliably identify their own limitations or know when to escalate to a "smart friend." A strong reviewer agent can't compensate for a weak primary. If quality is the goal, invest in the primary.

### The generator-evaluator anti-pattern (and fix)

Models reliably overpraise their own outputs — self-evaluation is biased. The fix: a GAN-inspired generator-evaluator split where a separate agent evaluates the generator's work against predetermined criteria rather than subjective judgment. For software: evaluator runs tests against the running application, not against the generator's self-report.

**Sprint contracts**: before implementation, generator and evaluator negotiate testable "done" definitions. This bridges the gap between high-level spec and verifiable outcome, and prevents the evaluator from grading on criteria the generator didn't know about.

Source: [Anthropic: Harness design for long-running application development (Mar 2026)](https://www.anthropic.com/engineering/harness-design-long-running-apps)

### What doesn't work

- **Unstructured swarms** — arbitrary agent negotiation networks with no clear ownership
- **Parallel writers** — agents making simultaneous writes to the same codebase
- **"Sexy" coordination patterns** — elaborate inter-agent protocols without single-threaded writes

### Test suites as primary communication (C compiler case study)

In a 16-agent parallel C compiler build (Anthropic, 2026), the test suite became the primary communication mechanism between agents. "Claude will solve whatever problem you give it — so the task verifier must be nearly perfect, otherwise Claude will solve the wrong problem." Invest as much in the verifier as in the task description.

Two additional operational lessons from this experiment:
- **Agent time blindness**: agents don't naturally sense elapsed time and can loop indefinitely. Use deterministic sampling modes and step limits to prevent endless testing cycles.
- **Monolithic task bottleneck**: when all agents hit the same indivisible problem, parallelism collapses. Decompose into independently solvable subtasks before assigning.

Source: [Anthropic: Building a C compiler with a team of parallel Claudes (Feb 2026)](https://www.anthropic.com/engineering/building-c-compiler)
