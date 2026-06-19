# How Real Products Are Built — Reference

Architectural analysis of 8 production agent products. Use it for pattern recognition, not exact implementation cloning. Product internals, benchmarks, and model routing change quickly; verify current docs/blogs before copying API details or numeric claims.

Evidence basis:
- Official product engineering posts and docs where available
- Open-source repositories for Aider, Cline, Bolt.new, and Priompt
- Third-party source-level analysis only where explicitly labeled

Benchmark and internals claims in this file are source-scoped. Treat vendor benchmarks as vendor claims and third-party reverse engineering as lower authority than official docs or open-source code.

## Table of Contents
1. [Claude Code](#claude-code)
2. [Cursor](#cursor)
3. [Aider](#aider)
4. [OpenAI Codex](#openai-codex)
5. [v0 by Vercel](#v0-by-vercel)
6. [Devin](#devin)
7. [Cline](#cline)
8. [Bolt.new](#boltnew)
9. [Cross-Cutting Patterns](#cross-cutting-patterns)

---

## Claude Code

**Architecture**: reactive tool loop with a large operational harness around it. "Minimal scaffolding, maximal harness."

**Secret sauce**: 5-layer compaction pipeline (budget reduction → snip → microcompact → context collapse → auto-compact). 7-layer permission system. Deferred tool schemas (only names loaded initially, full schemas via ToolSearch on demand).

**Key details**:
- 19 built-in tools + up to 35 conditional = up to 54 total tools
- Sub-agents run in isolated context, return only summary to parent
- Sessions stored as append-only JSONL files (auditability, recovery, fork/resume)
- Auto-compaction triggers at ~83.5% context utilization (~167K of 200K)
- Single `State` object overwritten completely at 7 "continue sites"
- Streaming via `StreamingToolExecutor` — tools execute as they stream in from model response
- Read-only operations (Read, Glob, Grep) execute in parallel; writes serialized

**Design philosophy**: No planning graphs, no state machines. The model reasons freely within a deterministic harness. Explicitly contrasted with LangGraph (explicit graph routing) or Devin (explicit task tracking).

Source: [Dive into Claude Code (arxiv)](https://arxiv.org/html/2604.14228v1). Treat source-level third-party analyses as lower authority than official docs; do not depend on proprietary or accidentally exposed code for production designs.

---

## Cursor

**Architecture**: Full VS Code fork (not a plugin). Three integrated layers.

**Secret sauce — Priompt**: JSX-based prompt compilation where each element has a priority score. When context exceeds limits, lower-priority elements are dropped via binary search. Open-sourced at `github.com/anysphere/priompt`.

**Secret sauce — Speculative Edits**: LLMs fail ~40% generating diffs, so Cursor does full-file rewrites with speculative decoding where the file being edited IS the draft. Achieves ~1,000 tokens/sec (13x speedup) on fine-tuned Llama-3-70b via Fireworks AI.

**Context engine**: Tree-sitter splits code at function/class boundaries. Merkle tree syncs with servers every ~5 minutes. Fine-tuned 7B CodeLlama reranker processes up to 500K tokens to select ~13K tokens from a million-line codebase.

**Tab prediction**: Online RL with +0.75 for accepted, -0.25 for rejected, 0 for silence. Retrains in 1.5-2 hours from 400M+ daily requests.

**Notable failure**: Shadow Workspace (hidden editor for AI linting) was retired — resource cost didn't justify the pipeline approach.

**Harness engineering (April 2026 blog post — Stefan Heule & Jediah Katz)**:

*Context evolution*: Early Cursor (late 2024) used heavy static context + guardrails (surfacing lint errors after every edit, rewriting file reads, limiting tool call count per turn). All of that is mostly gone now. As models improved, they knocked down guardrails and switched to minimal static context (OS, git status, current files) + dynamic context pulled by the agent on demand.

*Quality measurement*: Two complementary signals beyond benchmarks:
- **Keep Rate** — what fraction of agent-generated code remains in the codebase after fixed time intervals. If users manually fix or have the agent redo work, Keep Rate drops, indicating lower initial quality.
- **LLM semantic satisfaction** — a model reads the user's follow-up response to classify satisfaction: moving on to the next feature signals success; pasting a stack trace signals failure.

*Tool error classification*: Cursor classifies tool call errors by cause rather than treating all failures the same:
- `InvalidArguments` / `UnexpectedEnvironment` — model mistakes or context contradictions
- `ProviderError` — external vendor outage (GenerateImage, WebSearch)
- `UserAborted`, `Timeout` — expected workflow errors

Unknown errors are treated as bugs unconditionally. Expected errors use per-tool, per-model anomaly detection baselines — because different models error at different rates, a shared baseline would produce false alerts.

*Automated operations*: A weekly agent scans logs, surfaces new/spiked issues, and files tickets in Linear. Cloud Agents are triggered directly from Linear to kick off fixes. This "automated software factory" loop drove unexpected tool errors down by an order of magnitude over one focused sprint.

*Model-specific customization depth*: OpenAI models are trained on patch-based editing; Claude is trained on string replacement. Giving a model the "wrong" format costs extra reasoning tokens and produces more errors. Cursor provisions each model with the tool format it saw during training. Customization extends to prompting style: OpenAI models are more literal and precise in instruction following; Claude is more intuitive and tolerant of imprecise instructions. A model exhibiting **context anxiety** (refusing work as context fills up) can often be mitigated through prompt adjustments — harness-side workaround without model retraining.

*Mid-chat model switching*: When the user switches models, Cursor switches harnesses (different prompts, different tool set). Custom instructions tell the new model it's taking over mid-conversation and steer it away from tools it doesn't have. The conversation is summarized at switch time to reduce cache miss penalty. Recommendation: stay with one model per long task; use a fresh subagent instead if isolation is needed.

Sources: [cursor.com/blog](https://cursor.com/blog), [Fireworks: Cursor Fast Apply](https://fireworks.ai/blog/cursor), [Cursor: Continually improving our agent harness (Apr 2026)](https://cursor.com/blog/continually-improving-agent-harness)

---

## Aider

**Architecture**: Git-first terminal agent. Task-execute-commit-report.

**Secret sauce — PageRank Repo Mapping**: Tree-sitter parses into ASTs → directed graph (files as nodes, symbol references as edges) → NetworkX PageRank with personalization (active files get weight 100/N, default files get 1/N) → top results within token budget.

**Key details**:
- 130+ language support via Tree-sitter
- 5 edit formats: Whole, Diff, Diff-Fenced, Udiff, Editor-Diff (auto-selects per model)
- Architect Mode: planning model + execution model separation
- Reported lower token usage than more autonomous coding agents in some benchmarks; validate on your own workload
- Auto-lints and runs tests after every change, self-fixing detected issues
- `diskcache.Cache` avoids re-parsing unchanged files

Source: [aider.chat](https://aider.chat/), [github.com/Aider-AI/aider](https://github.com/Aider-AI/aider)

---

## OpenAI Codex

**Architecture**: Single-agent ReAct loop in `AgentLoop.run()`.

**Secret sauce — V4A Patch Format**: Custom diff format (`*** Begin Patch...*** End Patch`) that the model is specifically trained to excel at. The CLI intercepts patches, displays colorized diffs, and awaits approval.

**Two-phase sandbox**:
- Setup phase: network enabled (install dependencies)
- Agent phase: network disabled (security)
- macOS: Apple Seatbelt (`sandbox-exec`), Linux: bubblewrap + seccomp
- Protected paths: `.git`, `.agents`, `.codex`, `*.env` always read-only

**Shell-first philosophy**: The shell tool is the primary interface — Codex routes most actions through shell commands rather than specialized tools.

**Auto-review agent**: Separate reviewer model evaluates actions for data exfiltration, credential probing, persistence changes, and destructive operations before execution.

Source: [openai.com/index/unrolling-the-codex-agent-loop](https://openai.com/index/unrolling-the-codex-agent-loop/)

---

## v0 by Vercel

**Architecture**: Three-layer composite model for UI generation.

**Layer 1 — Dynamic Retrieval**: Context-aware prompt injection. Detects AI-related intent via embeddings + keywords, injects current framework docs.

**Layer 2 — LLM Suspense** (most innovative): Manipulates text during streaming before the user sees it. Long URLs replaced with proxies, icon mismatches fixed via vector search (100ms), imports corrected. User never sees intermediate incorrect state.

**Layer 3 — AutoFix**: Custom `vercel-autofixer-01` model trained via reinforcement fine-tuning. Runs 10-40x faster than gpt-4o-mini. 86.14% error-free. Dual approach: AST parsing for deterministic fixes + fine-tuned model for complex ones. Fixes execute in <250ms.

**Result**: v0-1.5-md composite achieves 93.87% error-free generation vs. 64.71% for raw Claude Sonnet 4.

Source: [vercel.com/blog/v0-composite-model-family](https://vercel.com/blog/v0-composite-model-family)

---

## Devin

**Architecture**: Cloud-based sandboxed VMs with shell, editor, and browser.

**Key learning**: "Senior-level at codebase understanding but junior at execution." Performs WORSE when given more instructions mid-task (unlike humans).

**Sweet spots**: Security fixes (20x efficiency), migrations (10-14x faster), test generation (50-60% → 80-90% coverage). PR merge rate: 34% → 67% over 18 months.

Source: [cognition.ai/blog/devin-annual-performance-review-2025](https://cognition.ai/blog/devin-annual-performance-review-2025)

---

## Cline

**Architecture**: VS Code extension (Apache 2.0). Plan-implement-verify-fix loop.

**Secret sauce**: Every action in Act mode requires explicit human approval. This gets it approved by enterprises. Code never leaves the approved environment. Open-source for security team auditing.

**Self-extending**: You can ask Cline to "add a tool" and it creates a new MCP server, configures and installs it.

Source: [github.com/cline/cline](https://github.com/cline/cline)

---

## Bolt.new

**Architecture**: Node.js entirely in the browser via WebContainers (WebAssembly).

**Runtime**: Rust-based filesystem in SharedArrayBuffer. Web Workers simulate processes. Custom TypeScript shell (JSH) emulates Bash. Service Workers intercept virtual localhost requests.

**Agent**: Single, carefully crafted LLM prompt generates complete application code. Stream parser detects artifacts and actions, ActionRunner executes in WebContainer.

**Limitation**: Cannot run native binaries — JS and WASM only.

Source: [github.com/stackblitz/bolt.new](https://github.com/stackblitz/bolt.new)

---

## Cross-Cutting Patterns

### Every Product Solves These Same Problems

1. **The Agent Loop**: All implement Think-Act-Observe. The differences are in how much autonomy the "Act" step gets.

2. **Context as Bottleneck**: Every product's most sophisticated engineering is about what to put in the context window. Solutions: Priompt priority (Cursor), PageRank (Aider), 5-layer compaction (Claude Code), RAG (Windsurf).

3. **Edit Application**: LLMs are bad at diffs. Solutions: speculative rewrites at 1000 tok/s (Cursor), V4A trained patches (Codex), search/replace blocks (Aider), streaming manipulation (v0).

4. **Sandboxing**: Universal concern. OS-level (Seatbelt, bubblewrap), VM-level (Firecracker), browser-level (WebContainers).

5. **Multi-agent/Parallel**: Running multiple agents on different parts of a codebase. Git worktrees (Cursor), isolated VMs (Devin), sub-agents (Claude Code).

### The Spectrum

- **Pair Programmers** (Aider, Codex CLI): Focused scope, lower cost profile, explicit review loop.
- **Orchestrators** (Claude Code, Cursor, Devin): Handle judgment calls across 40+ decisions. Higher cost but sustained context.

### The MCP Convergence
MCP is the clearest emerging interoperability standard for agent-to-tool communication. Many agent tools support it or are adding support, but production deployments still need auth, permissions, output limits, and observability around MCP calls.
