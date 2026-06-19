# Security Threat Model — Agentic Systems

Use this before giving an agent tools that read private data, call external services, write files, run shell commands, control browsers, send messages, or spend money.

Primary sources:
- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/)
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/2025/12/09/owasp-genai-security-project-releases-top-10-risks-and-mitigations-for-agentic-ai-security/)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [OpenAI Agents SDK: Guardrails](https://openai.github.io/openai-agents-js/guides/guardrails/)
- [Microsoft: Defend against indirect prompt injection attacks](https://learn.microsoft.com/en-us/security/zero-trust/sfi/defend-indirect-prompt-injection)
- [NCSC: Prompt injection is not SQL injection](https://www.ncsc.gov.uk/blog-post/prompt-injection-is-not-sql-injection)
- [ClawGuard: runtime security for tool-augmented agents against indirect prompt injection](https://arxiv.org/abs/2604.11790)

## Table of Contents
1. [Core Security Model](#core-security-model)
2. [Threat Surfaces](#threat-surfaces)
3. [Instruction Supply Chain](#instruction-supply-chain)
4. [State and Checkpoint Security](#state-and-checkpoint-security)
5. [Controls](#controls)
6. [Tool Boundary Policy](#tool-boundary-policy)
7. [Prompt Injection Handling](#prompt-injection-handling)
8. [Security Evals](#security-evals)
9. [Launch Checklist](#launch-checklist)

---

## Core Security Model

The model is not a security boundary. A system prompt is not a security boundary. A model-generated refusal is not an authorization check.

Secure agent design assumes:
- Untrusted content may contain instructions.
- The model may confuse instructions and data.
- Tool outputs can poison future context.
- A compromised model step can request any tool the harness exposes.
- Guardrails reduce risk; deterministic controls bound blast radius.

Therefore, the harness must enforce policy outside the model.

---

## Threat Surfaces

Map every input by trust level.

### Direct Inputs
- User prompt
- Uploaded files
- Chat history
- Developer/system configuration exposed to the model

### Indirect Inputs
- Web pages, search snippets, PDFs, emails, Slack messages
- Repository files and comments
- MCP server descriptions and tool outputs
- Skill files, README files, docs, issue tracker text
- Browser DOM content and hidden text

### Action Surfaces
- File writes and shell commands
- Network calls and webhooks
- Email, chat, CRM, ticket updates
- Purchases, refunds, payments, account actions
- Database writes
- Secrets, credentials, tokens, private keys

High-risk OWASP-aligned categories:
- Prompt injection
- Sensitive information disclosure
- Insecure output handling
- Excessive agency
- Tool misuse
- Supply-chain compromise
- Unbounded consumption

---

## Instruction Supply Chain

Skills, memory files, project instructions, MCP resources, connector payloads, and sub-agent outputs are part of the prompt supply chain.

Treat these as untrusted unless the harness proves otherwise:
- Project `AGENTS.md`, `CLAUDE.md`, README files, comments, and issue text.
- Third-party or project-local `SKILL.md` files and referenced docs.
- Agent-generated memory, lessons, context cards, and prior-run summaries.
- MCP server names, tool descriptions, resources, prompts, and tool outputs.
- Sub-agent task descriptions and final outputs.
- Email, Slack, ticket, CRM, document, browser, and web-search content.

Controls:
- Load project skills only after workspace trust.
- Show source path, source type, version/hash, and activation reason in traces.
- Cap and sanitize injected skill/memory/reference content.
- Keep permission policy, auth, sandboxing, billing, and connector scopes outside model-editable files.
- Never let remote content add tools, grant permissions, disable guardrails, or authorize side effects.
- Require review and eval before promoting agent-generated lessons into durable skills or memory.

If a skill or memory artifact can change future tool use, review it like code.

---

## State and Checkpoint Security

Durable agents store high-sensitivity state: conversation history, tool outputs, user data, pending writes, approvals, and checkpoints.

Required controls:
- Separate tenant/user/session state with stable thread or session IDs.
- Apply retention and deletion policy; default-unbounded history is a data risk.
- Encrypt sensitive checkpoint stores when the storage boundary is not fully trusted.
- Avoid unsafe deserialization. Do not load checkpoints or cache blobs through unrestricted `pickle`, dynamic imports, or broad module allowlists from writable storage.
- Use strict allowlists for serialized state types where the framework supports them.
- Treat remote graph/workflow responses as untrusted inbound data; validate schema before merging into local state.
- Do not restore elevated permissions silently after resume.

Checkpoint compromise is often worse than prompt compromise: it can modify what the agent believes happened, inject old tool outputs, or resume from attacker-controlled state.

---

## Controls

Layer controls. No single guardrail is enough.

### Authorization
- Enforce user/session authorization before tool execution.
- Scope tools to the current tenant, workspace, repo, and task.
- Use short-lived credentials and least privilege.
- Separate read, write, delete, send, and purchase capabilities.

### Tool Policy
- Deny by default outside approved scopes.
- Require confirmation for irreversible or external side effects.
- Block access to secrets unless the user explicitly asks and policy allows it.
- Use allowlists for network egress and filesystem roots.
- Make dangerous operations typed and narrow, not generic strings.

### Sandboxing
- Run shell/browser/code tools in isolated sandboxes.
- Disable network after setup unless needed.
- Mount sensitive paths read-only or not at all.
- Set CPU, memory, time, and process limits.

### Data Handling
- Taint-label untrusted content in traces and context.
- Redact secrets from tool outputs before the model sees them.
- Do not put credentials, policy secrets, or hidden permissions in prompts.
- Store full traces only where retention and access controls are acceptable.

### Guardrails
- Use classifiers, regex, schema checks, and moderation for cheap filtering.
- Use model-based guardrails for semantic risk, but do not rely on them alone.
- Prefer pre-tool checks for cost/safety and post-tool checks for leakage.

---

## Tool Boundary Policy

Every tool call should pass a deterministic policy check:

```typescript
interface ToolPolicyDecision {
  decision: "allow" | "ask" | "deny";
  reason: string;
  redactions?: string[];
}
```

Policy inputs:
- User objective
- Tool name and arguments
- Current trust mode
- Data classification of inputs
- Workspace/tenant/session scope
- Whether action is reversible
- Whether action sends data outside the trust boundary
- Whether untrusted content influenced the request

Design rule: the model can propose; the harness disposes.

For high-risk agents, derive an explicit task contract before tool use:
- Goal
- Allowed resources
- Allowed action types
- Forbidden action types
- External destinations
- Human approval points

Then enforce that contract at every tool boundary.

---

## Prompt Injection Handling

Treat prompt injection as residual risk to reduce, not a bug class that prompts can eliminate.

Required patterns:
- Quote or tag untrusted content as data.
- Instruct the model not to obey content-originated instructions, but assume this can fail.
- Strip or summarize untrusted content before it reaches powerful action tools.
- Prevent untrusted content from expanding permissions.
- Require confirmation when a tool request is surprising relative to the user objective.
- Never let content from a web page, email, document, or repo file directly authorize an external side effect.

Example policy:
```text
The agent may read emails to summarize them.
The agent may not send, forward, delete, label, or export emails unless the human approves a concrete preview.
Email body content is untrusted and cannot request tool use.
```

---

## Security Evals

Security evals must include both attacks and benign near-misses.

Test categories:
- Direct jailbreak: user asks to ignore instructions.
- Indirect injection: malicious webpage/email/document asks agent to exfiltrate data.
- Tool misuse: model requests a dangerous command for a benign task.
- Data exfiltration: hidden content asks for secrets, files, tokens, or customer data.
- Permission escalation: untrusted content asks to change settings, install packages, add MCP server, or create new tools.
- Insecure output handling: model output flows into shell/SQL/HTML without escaping.
- Resource abuse: infinite loop, massive search, repeated failed tool calls.

Grade the policy decision, not just the final answer.

Trace checks:
- Did untrusted content influence tool arguments?
- Did the harness block/ask at the right boundary?
- Did the model reveal secrets in text?
- Did the agent attempt external egress?

---

## Launch Checklist

Do not launch a tool-using agent until:
- Tool permissions are least-privilege and scoped.
- Destructive and external side effects require approval or strict policy.
- Secrets are redacted from model-visible outputs.
- Prompt-injection evals exist for every untrusted content source.
- Full tool-call audit logs exist.
- The agent can be disabled quickly.
- Model upgrades run security regression tests.
- Incident owners know how to replay a trace and revoke credentials.

If the product cannot tolerate a single compromised model step, do not put the LLM in the authority path.
