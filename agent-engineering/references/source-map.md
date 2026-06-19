# Source Map and Evidence Policy

Use this file to audit whether guidance in this skill rests on sources that are worth trusting. The goal is not to collect every article about agents; the goal is to keep a small, high-signal evidence base.

## Trust Tiers

### Tier 1 — Official API Docs and Specs

Use for exact implementation details: model IDs, SDK imports, method names, streaming helpers, tool-call formats, beta flags, pricing, limits, and auth.

- [Anthropic model docs](https://platform.claude.com/docs/en/about-claude/models/overview)
- [Anthropic Messages API / tool use docs](https://platform.claude.com/docs/en/docs/agents-and-tools/tool-use/overview)
- [Anthropic TypeScript SDK docs](https://platform.claude.com/docs/en/api/sdks/typescript)
- [Claude Agent SDK TypeScript docs](https://docs.claude.com/en/docs/agent-sdk/typescript)
- [Vercel AI SDK Agents docs](https://ai-sdk.dev/docs/agents/overview)
- [Vercel AI SDK Loop Control docs](https://ai-sdk.dev/docs/agents/loop-control)
- [OpenAI Agents SDK docs](https://openai.github.io/openai-agents-js/)
- [OpenAI Agents SDK Guardrails docs](https://openai.github.io/openai-agents-js/guides/guardrails/)
- [Model Context Protocol docs](https://modelcontextprotocol.io/)
- [OpenAI Codex CLI docs](https://developers.openai.com/codex/cli)
- [OpenAI Codex sandboxing and approvals docs](https://developers.openai.com/codex/agent-approvals-security)
- [OpenAI Codex configuration docs](https://developers.openai.com/codex/config-reference)
- [OpenAI Responses API local shell docs](https://developers.openai.com/api/docs/guides/tools-local-shell)
- [OpenAI Prompt Caching docs](https://developers.openai.com/api/docs/guides/prompt-caching)
- [Agent Skills specification](https://agentskills.io/specification)
- [Claude Code settings docs](https://code.claude.com/docs/en/settings)
- [Claude Code hooks docs](https://code.claude.com/docs/en/hooks)
- [Claude Code permissions docs](https://code.claude.com/docs/en/permissions)
- [Claude Agent SDK permissions docs](https://code.claude.com/docs/en/agent-sdk/permissions)
- [Apple App Sandbox docs](https://developer.apple.com/documentation/security/app_sandbox)
- [Apple App Sandbox entitlement reference](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/EnablingAppSandbox.html)
- [Apple: Embedding a command-line tool in a sandboxed app](https://developer.apple.com/documentation/Xcode/embedding-a-helper-tool-in-a-sandboxed-app)
- [Swift ArgumentParser](https://www.swift.org/blog/argument-parser/)
- [Swift Subprocess](https://github.com/swiftlang/swift-subprocess)

Rule: if Tier 1 disagrees with this skill, Tier 1 wins.

### Tier 2 — Official Engineering Guides From Builder Teams

Use for architectural principles and proven production patterns.

- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic: Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Anthropic: Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [OpenAI: Harness Engineering](https://openai.com/index/harness-engineering/)
- [OpenAI: Unrolling the Codex agent loop](https://openai.com/index/unrolling-the-codex-agent-loop/)
- [OpenAI: Unlocking the Codex harness / App Server](https://openai.com/index/unlocking-the-codex-harness/)
- [OpenAI GPT-5.2-Codex system card addendum](https://cdn.openai.com/pdf/ac7c37ae-7f4c-4442-b741-2eabdeaf77e0/oai_5_2_Codex.pdf)
- [Vercel: v0 composite model family](https://vercel.com/blog/v0-composite-model-family)

Rule: use these for design direction, but do not copy API snippets without checking Tier 1 docs.

### Tier 3 — Security Standards and Government Guidance

Use for security taxonomy, threat modeling, and launch gates.

- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/)
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/2025/12/09/owasp-genai-security-project-releases-top-10-risks-and-mitigations-for-agentic-ai-security/)
- [Microsoft: Defend against indirect prompt injection attacks](https://learn.microsoft.com/en-us/security/zero-trust/sfi/defend-indirect-prompt-injection)
- [Microsoft MSRC: How Microsoft defends against indirect prompt injection attacks](https://www.microsoft.com/en-us/msrc/blog/2025/07/how-microsoft-defends-against-indirect-prompt-injection-attacks/)
- [NCSC: Prompt injection is not SQL injection](https://www.ncsc.gov.uk/blog-post/prompt-injection-is-not-sql-injection)

Rule: prompts are never security boundaries. Use deterministic authorization, least privilege, sandboxing, and audit logs.

### Tier 4 — Papers, Preprints, and Open-Source Implementations

Use for emerging techniques, empirical evidence, and implementation inspiration. Treat preprints as useful but not definitive.

- [ClawGuard: runtime security for tool-augmented agents against indirect prompt injection](https://arxiv.org/abs/2604.11790)
- [Dive into Claude Code](https://arxiv.org/html/2604.14228v1)
- [Terminal Is All You Need](https://arxiv.org/abs/2603.10664)
- [Building Effective AI Coding Agents for the Terminal](https://arxiv.org/abs/2603.05344)
- [OpenAI Codex repository](https://github.com/openai/codex)
- [OpenAI Codex app-server README](https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md)
- [OpenAI Codex protocol notes](https://github.com/openai/codex/blob/main/codex-rs/docs/protocol_v1.md)
- [mini-SWE-agent project](https://github.com/SWE-agent/mini-swe-agent)
- [SWE-agent paper](https://arxiv.org/abs/2405.15793)
- [SWE-agent ACI notes](https://github.com/SWE-agent/SWE-agent/blob/main/docs/background/aci.md)
- [Aider project](https://github.com/Aider-AI/aider)
- [Aider repo map docs](https://aider.chat/docs/repomap.html)
- [Aider edit format docs](https://aider.chat/docs/more/edit-formats.html)
- [Cline project](https://github.com/cline/cline)
- [Bolt.new project](https://github.com/stackblitz/bolt.new)
- [microsoft/skills](https://github.com/microsoft/skills)
- [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents)
- [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)
- [lastmile-ai/mcp-agent](https://github.com/lastmile-ai/mcp-agent)
- [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness)
- [codejunkie99/agentic-stack](https://github.com/codejunkie99/agentic-stack)
- [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- [bytedance/deer-flow](https://github.com/bytedance/deer-flow)
- [wshobson/agents](https://github.com/wshobson/agents)
- [revfactory/harness](https://github.com/revfactory/harness)
- [Priompt project](https://github.com/anysphere/priompt)
- [Ratatui docs](https://ratatui.rs/)
- [Bubble Tea docs/repository](https://github.com/charmbracelet/bubbletea)

Rule: use open source for implementation patterns; use third-party source-level analysis for hypotheses, not hard guarantees.

### Tier 5 — Practitioner Essays and Independent Blogs

Use only for intuition, vocabulary, and idea generation.

- [Mihail Eric: The Emperor Has No Clothes](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)

Rule: never base numeric claims, API usage, or security controls on Tier 5 sources alone.

## Claim Rules

- **API/code claim**: must point to Tier 1 or official repository code.
- **Architecture claim**: should point to Tier 2, Tier 4 open-source code, or a clearly labeled product source.
- **Benchmark/number claim**: must name the source and scope. Vendor benchmarks are evidence, not universal truth.
- **Security claim**: must point to Tier 3 and should be enforced outside the model.
- **Product-internals claim**: label as official, open-source, or third-party analysis.
- **Heuristic**: if a recommendation is based on synthesis rather than a named source, label it as a heuristic and validate it with evals.

## Same-Day Verification List

Verify these against official docs on the day of implementation:

- Model IDs, aliases, context windows, max output, pricing, and rate limits
- SDK package names, imports, helper names, streaming methods, and beta flags
- Tool-call message formats and structured-output APIs
- Guardrail, approval, handoff, and hosted-tool behavior
- Prompt caching syntax, minimum token thresholds, TTL, and billing
- MCP transport/auth/security behavior
- OWASP/security guidance for new agentic threat categories
- Codex App Server protocol fields, event names, transports, approval request shapes, and version compatibility
- Codex sandbox mode semantics, protected paths, network rules, and macOS Seatbelt/sandbox-exec behavior
- Claude Code permission mode names, hook event schemas, settings precedence, and managed policy behavior
- Apple macOS sandbox/helper/XPC/security-scoped bookmark requirements for the target distribution path

## Local CLI Runner Source Pack

When designing a Claude Code/Codex-style local runner, prefer this order:

1. **Codex App Server + protocol** for the UI-client/local-core split, thread/turn/item primitives, streaming events, approvals, and stdio JSONL control plane.
2. **OpenAI local shell + Codex approvals/security/config docs** for shell execution ownership, sandbox/approval modes, network policy, and config trust.
3. **Claude Code permissions/settings/hooks** for permission precedence, shell-command matching pitfalls, lifecycle hooks, settings scopes, and sensitive file deny rules.
4. **SWE-agent + mini-SWE-agent** for ACI design, interactive/confirm/yolo modes, trajectories, and simple command-loop implementation.
5. **Aider docs** for repo-map context selection and model-facing edit formats.
6. **Apple App Sandbox entitlement docs** for Mac app embedding, user-selected file access, security-scoped bookmarks, child-process inheritance, XPC, network, and personal-data entitlements.
7. **OWASP/Microsoft guidance** for indirect prompt injection, tool misuse, least privilege, short-lived privileges, audit logging, and human review of risky actions.
8. **Terminal-agent papers** for research framing; treat fresh arXiv preprints as useful but not normative.

## Source Labels

When adding new claims, mentally tag them:

- `[official-doc]` exact API/spec behavior
- `[official-engineering]` production pattern from a builder team
- `[security-standard]` OWASP/government/vendor security guidance
- `[paper]` empirical or theoretical research
- `[open-source]` observed implementation
- `[vendor-claim]` public product benchmark or case study
- `[third-party-analysis]` independent source-level analysis
- `[heuristic]` synthesis that must be validated locally

If the tag would be `[heuristic]`, either keep the statement modest or add an eval requirement next to it.
