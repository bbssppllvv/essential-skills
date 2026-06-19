# Framework and API Watchlist

Use this before copying code examples or choosing models/frameworks. Agent APIs and model IDs change quickly.

Primary sources to verify:
- [Anthropic model docs](https://platform.claude.com/docs/en/about-claude/models/overview)
- [Claude Agent SDK TypeScript docs](https://docs.claude.com/en/docs/agent-sdk/typescript)
- [Vercel AI SDK docs](https://ai-sdk.dev/docs/foundations/agents)
- [OpenAI Agents SDK docs](https://openai.github.io/openai-agents-js/)
- [OpenAI API docs](https://platform.openai.com/docs/)
- [Model Context Protocol docs](https://modelcontextprotocol.io/)
- [OWASP GenAI Security](https://genai.owasp.org/)

## Table of Contents
1. [Always Verify](#always-verify)
2. [Anthropic / Claude](#anthropic--claude)
3. [Vercel AI SDK](#vercel-ai-sdk)
4. [OpenAI Agents SDK](#openai-agents-sdk)
5. [MCP](#mcp)
6. [Security Standards](#security-standards)
7. [Version Pinning](#version-pinning)

---

## Always Verify

Before implementation, verify:
- Current model IDs and aliases
- Context window and max output
- Tool calling message format
- Streaming event names and helpers
- Prompt caching syntax, TTL, minimum token thresholds, and pricing
- Guardrail/handoff semantics
- Human-in-the-loop APIs
- Beta headers and deprecation dates
- SDK package names and Node/Python version requirements
- Pricing and rate limits

Prefer exact model IDs in production. Aliases are fine for experiments but can silently change behavior.

---

## Anthropic / Claude

Verify in official docs:
- Latest models overview and pricing.
- Messages API tool-use format.
- Token counting API.
- Prompt caching requirements.
- Claude Agent SDK package and options.
- Claude Code tool names and permission modes.

Known volatile areas:
- Model family names and aliases.
- Extended/adaptive thinking settings.
- 1M context availability and beta flags.
- Tool runner beta APIs.
- Agent SDK package split and import paths.
- Cache TTL and billing fields.

Implementation rule: if a snippet uses a model ID, beta namespace, or cache setting, check docs on the day you ship.

---

## Vercel AI SDK

Verify in `ai-sdk.dev`:
- `generateText` / `streamText` parameters.
- `ToolLoopAgent` constructor and result types.
- `stopWhen`, `prepareStep`, `activeTools`, `toolChoice`.
- UI streaming helpers such as `toUIMessageStreamResponse` and `createAgentUIStreamResponse`.
- `tool`, `dynamicTool`, `toModelOutput`, approval APIs.
- Provider package versions.

Known volatile areas:
- V5 to V6 naming changes.
- UI stream protocol helpers.
- Agent abstraction names.
- Provider-specific options and caching.

Implementation rule: use AI SDK for web streaming and multi-provider ergonomics, but fall back to raw SDKs when provider-specific behavior matters.

---

## OpenAI Agents SDK

Verify in official JS/Python docs:
- Package install and runtime requirements.
- `Agent`, `run`, `Runner`, sessions, streaming.
- Handoffs vs agents-as-tools.
- Guardrails: input, output, and tool guardrails.
- Hosted tools and whether they support the same guardrail path as function tools.
- Tracing and eval integrations.

Known volatile areas:
- Python and TypeScript feature parity.
- Built-in sandbox/code-mode availability.
- Hosted tools behavior on non-OpenAI providers.
- Guardrail execution order and handoff boundaries.

Implementation rule: use it for OpenAI-first agents, voice agents, handoffs, and tracing. Be cautious when mixing non-OpenAI providers with hosted OpenAI tools.

---

## MCP

Verify:
- Transport support: stdio, HTTP, SSE/streamable HTTP.
- Auth model and credential storage.
- Tool naming conventions.
- Resource/list/read support.
- Output token limits.
- Client confirmation requirements.
- Server trust level.

Known risk: MCP server descriptions and outputs are model-visible content. Treat third-party MCP servers as code plus prompt surface.

Implementation rule: wrap MCP tools with the same permission, redaction, logging, and output-limiting policy as first-party tools.

---

## Security Standards

Track:
- OWASP Top 10 for LLM Applications.
- OWASP Agentic AI / GenAI Security project materials.
- Provider guidance on prompt injection and tool safety.
- Browser/computer-use security advisories.

Known volatile areas:
- Prompt injection mitigations.
- Agent-to-agent protocols.
- Browser agent security.
- Tool marketplace / MCP supply-chain risks.

Implementation rule: prompts reduce risk; deterministic tool-boundary controls limit impact.

---

## Version Pinning

Record these in every production trace:
- Agent version / git SHA
- Prompt version
- Tool schema version
- Tool implementation version
- Model ID and provider
- SDK package versions
- Runtime version
- Eval dataset version

When upgrading:
1. Read migration docs.
2. Run eval suite.
3. Run security suite.
4. Canary.
5. Watch cost, latency, tool failures, safety denials, and user escalations.
