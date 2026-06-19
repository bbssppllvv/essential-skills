---
name: agent-engineering
description: >
  Build production-grade AI agents and agentic workflows: agent loops, tool calling,
  Agent-Computer Interface (ACI) design, harness/scaffolding, context engineering,
  error recovery, permissions, security threat modeling, multi-agent orchestration,
  cost controls, eval harnesses, production runbooks, and API/version watchlists. Use
  when designing or implementing LLM agents, coding assistants,
  Claude Code/Codex/Cursor/Devin-style systems, ReAct loops, tool-use workflows,
  Claude Agent SDK, Vercel AI SDK ToolLoopAgent, OpenAI Agents SDK, or any system
  where an LLM reasons and acts through tools.
---

# Agent Engineering

A research-backed guide to building AI agents. Synthesized from Anthropic's official engineering guides, Claude Code / Codex-style harness patterns, Vercel AI SDK v6, OpenAI Agents SDK, and public analysis of production agent products.

**Use this as engineering guidance, not an immutable specification.** Prefer official SDK docs for exact API names, model IDs, prices, and beta flags. Treat product-specific numbers below as evidence for design trade-offs, not reusable constants.

**Evidence policy**: every nontrivial recommendation should trace to official docs, official engineering writeups, security standards, papers, open-source implementations, or clearly labeled practitioner heuristics. See `references/source-map.md` before adding new claims or relying on numbers.

**Useful intuition, not authority**: [Mihail Eric, "The Emperor Has No Clothes"](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/) — agents can be surprisingly small at the decision-loop level; production quality comes from the harness around that loop.

---

## The Core Insight

> "The LLM never actually touches your filesystem. It just *asks* for things to happen, and your code makes them happen."

Every AI agent follows the same architecture. The LLM is a brain in a jar — it can think and talk, but cannot act. Your code (the **harness**) is the body — it receives structured requests, executes them, and feeds results back. The LLM is interchangeable; the harness is your product.

**Anthropic's guiding principle**: "Find the simplest solution possible, and only increase complexity when needed. This might mean not building agentic systems at all."

---

## Anthropic's Taxonomy: Workflows vs. Agents

Before building, understand what you actually need:

- **Workflows**: LLMs orchestrated through **predefined code paths**. You design the control flow; the LLM fills in steps. (Prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer.)
- **Agents**: LLMs **dynamically direct their own processes and tool usage**. The model decides what to do next based on what it observes.

Most real-world systems are workflows, not agents. Only use agents for open-ended problems where the number of steps is unpredictable. See `references/anthropic-patterns.md` for the 6 building blocks.

---

## MVP Boundary First

When asked to build, design, or audit an agent, start with a small harness boundary rather than a full autonomy platform. Capture these before architecture:

1. **Domain and job-to-be-done** — what work the agent performs and what output is useful.
2. **Autonomy level** — answer-only, draft-only, approval-gated action, policy-bounded action, or long-running goal worker.
3. **Risk level** — read-only, internal write, external communication, financial, legal/health/safety, security-sensitive, destructive, or privileged.
4. **State duration** — single turn, multi-turn session, resumable workflow, or long-running objective.
5. **Tool surface** — internal APIs, MCP/connectors, browser, shell/filesystem, database, documents, messaging, payments, or compute sandbox.
6. **Validation signal** — the deterministic or reviewable evidence that proves the task is complete.

Default to the lowest autonomy level that produces value. For most business agents, the first useful version is draft-only or approval-gated. Add goal loops, broad connector access, and sub-agents only after the single-agent MVP fails measurable evals for reasons those features solve.

Recommended build order:

```text
manual loop -> typed tools -> permission engine -> structured observations
-> step/cost budgets -> traces -> planning mode -> durable state
-> retrieval/scoped instructions -> compaction -> governed skills/connectors
-> long-running goals -> sub-agents
```

Use `references/agent-type-recipes.md` for domain defaults and `references/production-runbook.md` before launch.

---

## The Agent Loop

The heartbeat of every agent — a `while` loop where the LLM decides to call tools or respond:

```
OUTER LOOP (user turn)
│  User sends a message → append to conversation
│
│  INNER LOOP (agent turn)
│  │  Send conversation to LLM
│  │  LLM responds with text and/or tool calls
│  │  If tool calls: execute, append results, continue
│  │  If no tools: show response, break to outer loop
```

This is the architectural core. Claude Code, Codex-style CLIs, Cursor-like IDE agents, and Devin-like cloud agents differ mostly in the harness: permissions, context assembly, tool execution, observability, persistence, evals, and UX. A 2026 public source-level analysis of Claude Code describes the core as a simple tool loop with most system complexity in the surrounding infrastructure.

### Minimal Implementation (TypeScript, Anthropic SDK)

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "Fix the bug in auth.ts" },
];

let response = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 4096,
  tools,
  messages,
});

while (response.stop_reason === "tool_use") {
  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const block of response.content) {
    if (block.type === "tool_use") {
      try {
        const result = executeTool(block.name, block.input);
        toolResults.push({ type: "tool_result", tool_use_id: block.id, content: String(result) });
      } catch (err) {
        toolResults.push({ type: "tool_result", tool_use_id: block.id, content: String(err), is_error: true });
      }
    }
  }
  messages.push({ role: "assistant", content: response.content });
  messages.push({ role: "user", content: toolResults });
  response = await client.messages.create({ model: "claude-sonnet-4-6", max_tokens: 4096, tools, messages });
}
```

### With Vercel AI SDK (higher-level)

```typescript
import { ToolLoopAgent, tool, stepCountIs } from "ai";
import { anthropic } from "@ai-sdk/anthropic";
import { readFileSync } from "node:fs";
import { z } from "zod";

const agent = new ToolLoopAgent({
  model: anthropic("claude-sonnet-4-6"),
  instructions: "Work carefully, explain risky changes, and verify before finishing.",
  tools: {
    readFile: tool({
      description: "Read a file at the given path",
      inputSchema: z.object({ path: z.string() }),
      execute: async ({ path }) => readFileSync(path, "utf-8"),
    }),
  },
  stopWhen: stepCountIs(10),
});

const { text, steps } = await agent.generate({
  prompt: "Fix the bug in auth.ts",
});
```

### With Claude Agent SDK (production-grade)

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Fix the bug in auth.ts",
  options: { allowedTools: ["Read", "Edit", "Bash"] },
})) {
  console.log(message);
}
```

See `references/agent-loop.md` for loop variants (plan-then-execute, iterative refinement, human-in-the-loop) and `references/frameworks.md` for framework-specific patterns.

For low-level local CLI/TUI runners that operate directly on a Mac or developer workstation, read `references/local-cli-runners.md` before designing. It is source-backed from Codex App Server/protocol docs, Claude Code permission/hook docs, SWE-agent/mini-SWE-agent, Aider, Apple App Sandbox docs, OWASP/Microsoft security guidance, and terminal-agent papers. It covers the local core + UI-client split, shell execution, PTYs, patch application, macOS sandboxing, workspace trust, approvals, event logs, replay, and TUI/GUI ergonomics.

---

## The Three Layers

### Layer 1: The LLM (the brain)

Controlled through system prompt, conversation history, and tool definitions. The LLM never executes anything — it only produces structured output: text for the user, or tool calls for the harness.

### Layer 2: Tools / ACI (the hands)

Anthropic considers tool design as important as prompt engineering — they call it **ACI (Agent-Computer Interface)**:

> "Think about how much effort goes into human-computer interfaces, and plan to invest just as much effort in creating good agent-computer interfaces."

**Five principles from production:**

1. **Descriptions are API docs for the LLM.** Include what, when, constraints, edge cases, examples. Anthropic's SWE-bench team "spent more time optimizing tools than the overall prompt."

2. **Return rich, structured results.** Not "success" — return what changed, current state, a preview the model can verify. For errors: what went wrong, similar matches, recovery suggestions.

3. **Fail informatively.** Return errors as tool results with `is_error: true`, not exceptions. The LLM is good at recovering if told what went wrong.

4. **Keep tools atomic.** One tool, one action. `read_and_edit_file` is worse than separate tools.

5. **Mistake-proof (poka-yoke).** Anthropic changed SWE-bench tools to require absolute file paths — "the model used this method flawlessly." Use enums, reasonable defaults, derive what you can.

**Tool count matters**: every active tool competes for attention and context. Keep the default active set small, group rarely used capabilities behind tool search/deferred schemas, and confirm the threshold with evals. Large products can expose many tools by activating only the relevant subset at each step.

See `references/tool-design.md` for schemas, registry patterns, and anti-patterns.

### Layer 3: The Harness (the body)

Everything around the loop: streaming, context management, safety, persistence, cost tracking, evals, and UX. This is where most production engineering effort goes.

See `references/harness.md` for production patterns.

---

## Context Engineering

The discipline of curating the optimal set of tokens during inference. This is a critical skill for agent builders.

**Key finding (JetBrains Research / TUM, 2025)**: On SWE-agent / SWE-bench Verified, observation masking (hiding older environment observations while preserving recent turns) matched or beat LLM summarization in 4/5 tested settings and cut cost substantially in the reported configurations. This supports trying cheap masking/truncation before model-generated summaries, but the masking window must be tuned per harness.

### Strategies (ordered by cost)

1. **Truncate old tool results** — old file contents rarely needed verbatim. Cheapest, safest.
2. **Sliding window** — keep system prompt + last N turns. Simple, predictable.
3. **Compaction** — delete low-value tokens (every surviving line is verbatim). No hallucination.
4. **Summarize-and-drop** — use a cheaper model to preserve decisions and task state when masking/compaction is no longer enough.

### Claude Code-Style 5-Layer Compaction Pipeline

Public source-level analysis of Claude Code describes 5 sequential shapers before model calls:
1. **Budget reduction** — per-message size limits on tool results
2. **Snip** — lightweight temporal trimming of older segments
3. **Microcompact** — fine-grained compression (time-based + cache-aware)
4. **Context collapse** — read-time projection with summaries stored separately
5. **Auto-compact** — full model-generated summary (only when layers 1-4 fail)

**Trigger point**: one reported implementation auto-compacts around high context utilization. Do not hardcode the exact percentage; tune thresholds against latency, cache behavior, and task success.

### Just-in-Time Context

Don't load everything upfront. Agents maintain lightweight references (file paths, URLs) and load content dynamically via tools at runtime. Claude Code uses lazy CLAUDE.md loading and deferred tool schemas for this reason.

### Exception: stable domain knowledge belongs in the system prompt

Counter-intuitive finding from Vercel evals (Jan 2026): for stable, domain-specific documentation, **always-available context in the system prompt outperforms on-demand skill loading** — significantly.

| Configuration | Pass rate |
|---------------|-----------|
| No documentation | 53% |
| Skill (not loaded) | 53% |
| Skill + explicit "load this first" instructions | 79% |
| Compressed docs in system prompt (AGENTS.md style) | **100%** |

Why: on-demand loading creates **decision points** — the agent must recognize it needs the skill, decide to load it, and sequence that correctly. Each decision point is a failure mode. Docs always present in the system prompt eliminate the decision entirely.

The practical pattern: compress stable documentation (Vercel reduced 40KB → 8KB via pipe-delimited index) and embed it directly. For dynamic content (live data, user files, search results) — keep just-in-time. For stable domain knowledge — put it upfront.

Instruction that helps: *"Prefer retrieval-led reasoning over pre-training-led reasoning"* — tells the agent to consult the provided docs rather than rely on training-time knowledge.

Source: [Vercel: AGENTS.md outperforms skills in our agent evals (Jan 2026)](https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals)

See `references/harness.md` for implementation details and the compaction code patterns.

---

## Safety & Permissions

### Claude Code-Style 7-Layer System

Public analysis describes layered controls for tool use. The product-specific details vary, but the design principle is stable: independent checks should all have the ability to block unsafe action.
1. **Hooks** — PreToolUse callbacks (allow/deny/pass)
2. **Deny rules** — always override allow, even in bypassPermissions
3. **Permission mode** — graduated trust (plan → default → acceptEdits → auto → bypassPermissions)
4. **Allow rules** — user-configured auto-approvals
5. **canUseTool callback** — runtime approval prompt
6. **Shell sandboxing** — filesystem/network isolation
7. **Session non-restoration** — permissions not restored on resume (anti-replay)

### The Permission Spectrum

```
Reads: usually allow inside the approved workspace, but redact or block secrets
Searches: usually allow inside approved scopes
Edits: require review (user sees diff)
Shell: filtered (safe auto-approved, dangerous need confirmation)
Destructive: always ask (rm -rf, git push --force, DROP TABLE)
```

Anthropic data: users approve ~93% of permission prompts, indicating approval fatigue. The solution is automated boundaries, not relying on user vigilance.

---

## Production Reality Check

Production lessons that repeatedly show up across agent systems:

- **Expect nonzero failure rates.** Agents are stochastic, stateful systems. Define what failure means for your task and measure it.
- **Large active tool sets degrade behavior.** Prefer dynamic activation, tool search, or narrower agent roles over exposing everything.
- **Multi-agent is expensive.** Anthropic reported multi-agent research systems using about 15x chat-token usage and excelling mainly on breadth-first research. Relative cost versus a single agent depends on workload, but "several-x more" is the safe planning assumption.
- **Context fails by content, not just by clock time.** Long sessions degrade when old tool outputs, stale plans, and irrelevant observations dominate the context. Trigger compaction by token budget and eval behavior, not wall-clock alone.
- **Agent-legible environments compound.** If policies, plans, logs, metrics, UI state, quality gates, or source-of-truth facts are not retrievable or inspectable through approved tools, the agent cannot reliably use them. Put durable knowledge in versioned artifacts and expose validation signals directly.
- **Latency UX matters.** Stream text, stream tool progress, and surface checkpoints so the user can tell whether the agent is working or stuck.

### Cost Optimization

1. **Route by difficulty** — cheap/fast models for classification, formatting, extraction, simple sub-agent work; stronger models for planning, synthesis, and risky edits.
2. **Prompt caching** — cache stable system prompts, tool definitions, and long static context where the provider supports it.
3. **Context compaction** — mask/truncate old observations before summarizing with another model.
4. **Limit work** — max turns, max tool calls, per-tool timeouts, total session budget, stuck-loop detection.
5. **Batch when latency is not user-facing** — async jobs, eval runs, and offline processing can trade latency for lower cost.

See `references/harness.md` § Cost Controls for implementation.

---

## Multi-Agent Architecture

Anthropic's research system with Opus as lead + Sonnet sub-agents **outperformed single-agent Opus by 90.2% on an internal research eval**. Token usage explained 80% of the measured variance. This is strong evidence for breadth-first research, not a blanket argument for multi-agent coding.

But multi-agent is expensive and complex. Only use when you need:
- **Parallel independent work** (review 50 files simultaneously)
- **Context isolation** (prevent one subtask from polluting another)
- **Specialized roles** (different system prompts, tool sets)

**Production default**: start with a single tool-using agent or a workflow. Add sub-agents only when they buy parallelism, context isolation, or specialization that evals can see.

See `references/multi-agent.md` for patterns (orchestrator+workers, handoffs, pipeline, swarm) and framework-specific implementations.

---

## Evaluation

From Anthropic's "Demystifying Evals" guide:

- **Start with 20-50 tasks** from real failures — don't wait for hundreds
- **Grade outcomes, not paths** — don't specify exact tool sequences
- **Three grader types**: code-based (fast, objective), model-based (flexible), human (gold standard)
- **pass@k** — probability of 1+ correct in k attempts (for development)
- **pass^k** — probability ALL k succeed (for customer-facing reliability)
- **Read transcripts** — when scores don't climb, it's often the eval, not the agent

For a production-grade eval suite, use `references/eval-harness.md`: it covers dataset shape, deterministic/model/human graders, pass@k vs pass^k, trace schemas, release gates, and common eval bugs.

---

## The 6-Layer Harness Taxonomy

A production agent harness has roughly 6 independent layers, each of which must be optimized. If any one degrades, the whole agent experience degrades.

| Layer | Responsibility |
|-------|---------------|
| **Orchestration** | Determines which steps the system should take |
| **Context** | Assembles instructions and information to steer the model |
| **Inference routing** | Selects which AI provider fulfills the request |
| **Transport** | Streams messages reliably to and from AI providers |
| **State** | Persists conversations and enables resumption |
| **Execution** | Lets the agent interact with the environment via tool calls |

Most public writing focuses on Context and Execution. Inference routing, Transport, and State are underinvested and often the source of subtle production failures.

Source: [Cursor — Continually improving our agent harness (Apr 2026)](https://cursor.com/blog/continually-improving-agent-harness)

---

## Mental Models

### "The harness is the product"
The LLM is replaceable more often than the product feels like it is. The harness — how you orchestrate the loop, manage context, handle permissions, surface progress, and verify work — is what makes your agent reliable.

### "Third-party harnesses can outperform first-party"
A common misconception: first-party harnesses from the model labs (Anthropic, OpenAI, Google) will always outperform third-party ones. Specialization wins. A harness tuned to a specific use case — with model-specific tool formats, tailored prompting styles, custom eval feedback loops, and aggressive context optimization — routinely beats generic API access to the same model. The lab ships a model; the harness builder ships a product. (Evidence: Cursor's tuned harness outperforms direct API access on coding tasks. Aider auto-selects edit formats per model for the same reason.)

### "Context is the binding constraint"
Not compute, not memory — context window capacity determines what an agent can do. Every architectural decision (lazy loading, deferred schemas, sub-agent isolation, compaction) reflects this.

### "ACI is the new HCI"
Tool design is as important as UI design. The LLM is your API consumer. Invest accordingly.

### "Simple beats clever"
The core loop can be small; the product quality comes from the harness. Start simple, add complexity only when you hit real problems that show up in evals or production traces.

---

## Production Kit

Use these references before claiming an agent is production-ready:

1. **Threat model first** — `references/security-threat-model.md` before granting tools that read private data, write files, run shell, browse authenticated pages, send messages, or spend money.
2. **Eval harness before tuning** — `references/eval-harness.md` before changing prompts, model IDs, tool schemas, or orchestration.
3. **Runbook before launch** — `references/production-runbook.md` for observability, state, rollback, incident response, cost controls, and model-upgrade process.
4. **Recipe before architecture** — `references/agent-type-recipes.md` to choose workflow vs agent vs multi-agent by task shape.
5. **Legibility before autonomy** — `references/agent-legible-environments.md` to make source-of-truth knowledge, validation signals, and feedback loops inspectable before raising autonomy.
6. **Govern skills/connectors before scaling them** — `references/skills-connectors-governance.md` before installing project skills, marketplace skills, MCP servers, plugins, or agent-generated lessons.
7. **Watchlist before copying code** — `references/framework-api-watchlist.md` because model IDs, SDK helper names, streaming APIs, guardrails, and beta flags change quickly.

---

## How Real Products Do It

| Product | Secret Sauce | Architecture |
|---------|-------------|-------------|
| **Claude Code** | 5-layer compaction, 7-layer permissions, deferred tool schemas | Single-threaded reactive loop, sub-agents with isolated context |
| **Cursor** | Priompt priority context, model-specific tool formats, Keep Rate + LLM satisfaction evals, per-model anomaly detection | VS Code fork, codebase indexing, per-model harness customization |
| **Aider** | Repo map + git-native workflow | Terminal agent, edit formats tuned per model |
| **OpenAI Codex** | App Server protocol, event-streamed turns/items, patch/application flow, sandbox + approval policies | Local core that can be driven by CLI/TUI/IDE/desktop clients; platform-specific subprocess sandboxing |
| **v0** | Retrieval + streaming correction + fast autofix loop | Composite UI-generation pipeline |
| **Devin** | Cloud-based sandboxed VMs, interactive planning | Full development environment per agent instance |

See `references/real-products.md` for detailed architectural breakdowns.

---

## References

| File | When to read |
|------|-------------|
| `references/source-map.md` | Source hierarchy, evidence policy, trust tiers, and rules for adding claims |
| `references/anthropic-patterns.md` | Anthropic's building blocks, long-running harness, eval methodology, Claude Agent SDK |
| `references/agent-loop.md` | Loop variants (ReAct, plan-execute, iterative), conversation management, thinking patterns |
| `references/tool-design.md` | ACI principles, JSON schemas, tool registry, toModelOutput, anti-patterns |
| `references/agent-legible-environments.md` | Source-of-truth artifacts, validation signals, mechanical invariants, and entropy cleanup |
| `references/skills-connectors-governance.md` | Skill package contracts, trigger evals, trust tiers, MCP/connector governance, self-improvement promotion |
| `references/local-cli-runners.md` | Build Claude Code/Codex-style local CLI runners: shell, patches, macOS sandboxing, approvals, traces |
| `references/harness.md` | Streaming (Anthropic SDK, AI SDK, SSE), compaction, safety, persistence, cost, deployment |
| `references/eval-harness.md` | Production eval design, datasets, graders, metrics, trace schemas, release gates |
| `references/security-threat-model.md` | Prompt injection, tool-boundary policy, permissions, secrets, sandboxing, security evals |
| `references/production-runbook.md` | Observability, reliability, persistence, deployment, cost management, incident response |
| `references/agent-type-recipes.md` | Architecture recipes for coding, research, browser, support, data, docs, SRE, automation agents |
| `references/framework-api-watchlist.md` | Current API/model/version checklist for Anthropic, Vercel AI SDK, OpenAI Agents SDK, MCP, OWASP |
| `references/multi-agent.md` | Orchestrator+workers, handoffs, pipeline, swarm, coordination, production cost data |
| `references/real-products.md` | Architecture details for Cursor, Aider, Devin, Codex, v0, Bolt.new, Cline, Windsurf |
| `references/frameworks.md` | Vercel AI SDK v6, Claude Agent SDK, OpenAI Agents SDK, Mastra, LangGraph, LlamaIndex |

---

## Key Sources

- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic: Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic: Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Multi-Agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Anthropic: Demystifying Evals](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [OpenAI: A Practical Guide to Building Agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [OpenAI: Harness Engineering](https://openai.com/index/harness-engineering/)
- [OpenAI: Unrolling the Codex Agent Loop](https://openai.com/index/unrolling-the-codex-agent-loop/)
- [OpenAI: Unlocking the Codex Harness / App Server](https://openai.com/index/unlocking-the-codex-harness/)
- [OpenAI Codex CLI](https://developers.openai.com/codex/cli)
- [OpenAI Codex App Server README](https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md)
- [OpenAI Responses API Local Shell](https://developers.openai.com/api/docs/guides/tools-local-shell)
- [OpenAI GPT-5.2-Codex System Card Addendum](https://cdn.openai.com/pdf/ac7c37ae-7f4c-4442-b741-2eabdeaf77e0/oai_5_2_Codex.pdf)
- [Claude Code Permissions](https://code.claude.com/docs/en/permissions)
- [Claude Code Hooks](https://code.claude.com/docs/en/hooks)
- [OpenAI Agents SDK](https://openai.github.io/openai-agents-js/)
- [Claude Code Architecture (arxiv)](https://arxiv.org/html/2604.14228v1)
- [Agent Skills specification](https://agentskills.io/specification)
- [microsoft/skills](https://github.com/microsoft/skills)
- [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents)
- [lastmile-ai/mcp-agent](https://github.com/lastmile-ai/mcp-agent)
- [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness)
- [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)
- [bytedance/deer-flow](https://github.com/bytedance/deer-flow)
- [mini-SWE-agent](https://mini-swe-agent.com/latest/usage/mini/)
- [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://arxiv.org/abs/2405.15793)
- [Aider repo map](https://aider.chat/docs/repomap.html)
- [Aider edit formats](https://aider.chat/docs/more/edit-formats.html)
- [Apple App Sandbox Entitlement Reference](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/EnablingAppSandbox.html)
- [Terminal Is All You Need](https://arxiv.org/abs/2603.10664)
- [Building Effective AI Coding Agents for the Terminal](https://arxiv.org/abs/2603.05344)
- [Vercel AI SDK 6](https://ai-sdk.dev/docs/foundations/agents)
- [Claude Agent SDK (TypeScript)](https://docs.claude.com/en/docs/agent-sdk/typescript)
- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/)
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/2025/12/09/owasp-genai-security-project-releases-top-10-risks-and-mitigations-for-agentic-ai-security/)
- [Mihail Eric: The Emperor Has No Clothes](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)
