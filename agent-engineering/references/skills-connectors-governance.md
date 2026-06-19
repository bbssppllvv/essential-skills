# Skills and Connectors Governance

Use this before adding skill packages, project instructions, MCP servers, plugins, connector tools, memory files, or agent-generated lessons to an agent harness.

Evidence basis:
- [Agent Skills specification](https://agentskills.io/specification)
- [microsoft/skills](https://github.com/microsoft/skills)
- [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents)
- [lastmile-ai/mcp-agent](https://github.com/lastmile-ai/mcp-agent)
- [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness)
- [codejunkie99/agentic-stack](https://github.com/codejunkie99/agentic-stack)
- [bytedance/deer-flow](https://github.com/bytedance/deer-flow)
- [wshobson/agents](https://github.com/wshobson/agents)
- [revfactory/harness](https://github.com/revfactory/harness)

## Core Rule

Skills and connectors are executable influence, not passive documentation. They change what the model sees, which tools it can discover, which policies it believes apply, and what outputs re-enter the main context.

Treat every skill, memory file, connector response, MCP server description, and sub-agent output as an instruction-supply-chain dependency with provenance, activation rules, evals, and rollback.

## Skill as Runtime Artifact

Do not evaluate a skill like documentation. Evaluate it like a runtime artifact that can change agent behavior.

Minimum production contract:

```text
activation:
  trigger hint
  negative triggers
  conflict skills
  lazy references/scripts/assets

authority:
  owner
  source / trust tier
  version or content hash
  allowed tools/resources
  forbidden policy changes

verification:
  expected artifact or behavior
  validation signal
  eval cases
  known failure modes
```

Keep the default loaded metadata small. Load the skill body only when the trigger matches, and load references/scripts/assets only when the body says they are needed.

Match the degree of freedom to the task:
- **High freedom**: broad judgment, design principles, review checklists.
- **Medium freedom**: workflows, pseudocode, examples, common pitfalls.
- **Low freedom**: fragile deterministic work; provide scripts, schemas, validators, or commands.

Good skills describe destinations and fences: outcome, constraints, source-of-truth files, validation signal, and hazards. Avoid brittle step-by-step instructions unless the exact sequence is the product.

## Activation Quality

Skill activation is a classifier. Bad activation silently degrades the agent: false negatives lose expertise, false positives inject irrelevant instructions, and overlapping triggers create nondeterministic context.

Test:
- True positives: prompts that should load the skill.
- False positives: similar prompts that should not load it.
- False negatives: paraphrases, abbreviations, and realistic user wording.
- Conflict cases: two skills with overlapping triggers.
- Regression cases: old tasks after the trigger description changes.

Keep an index or manifest that makes duplicate triggers visible. If two skills claim the same intent, either merge them, make one a router, or narrow their descriptions.

For large domains, use a root/router skill:
- Root skill stays small.
- Root maps user intent to one or more workflow-specific subskills.
- Root forces the exact subskill load before action.
- Subskills own detailed API, platform, or workflow instructions.

## Trust Tiers

Minimum trust model:

```text
built_in       maintained with the harness
user           installed by the user
project        loaded from the current repository or workspace
third_party    installed from a marketplace or GitHub repo
generated      produced by an agent from prior runs
remote         returned by MCP/connectors/sub-agents at runtime
```

Rules:
- Built-in and user skills can be enabled by default if the install path is trusted.
- Project skills are disabled until workspace trust is established.
- Third-party skills require source review, pinning, and an uninstall path.
- Generated skills are candidates only; they are not active until review and eval pass.
- Remote content is data, never policy.

Every loaded skill should be inspectable by source path and version. The agent should be able to answer "why was this skill active?" from trace data.

## Instruction Supply Chain Risks

High-risk sources:
- `SKILL.md` bodies and referenced docs
- project `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, README files, issue text
- memory/context-card files
- MCP server names, descriptions, prompts, resources, and tool outputs
- sub-agent task descriptions and sub-agent final outputs
- connector payloads such as email, Slack, tickets, CRM, docs, and web pages

Controls:
- Tag origin and trust tier in traces and context.
- Cap skill/reference/memory sizes before model injection.
- Preserve source paths and content hashes.
- Keep permission policy outside skills and project docs.
- Prevent skills or remote content from adding tools, changing auth, granting permissions, or disabling guardrails.
- Sanitize or quote untrusted content before it can influence side-effecting tools.

## MCP and Connector Rules

MCP/connectors need the same governance as local tools plus network and tenancy boundaries:

- Namespace tools by server and connector to prevent name shadowing.
- Pin allowed servers, transports, and resource roots.
- Separate authentication from authorization: a connected account does not imply every action is allowed.
- Apply the normal tool contract: risk class, side-effect class, timeout, output cap, retry policy, audit policy.
- Isolate request/session context so logs, notifications, resources, auth state, and temporary tool results cannot bleed between clients.
- Treat remote tool descriptions and resource contents as untrusted data unless the server is part of the trusted harness.
- Record the connector account, tenant, scopes, and permission decision for every side-effecting call.

When exposing an agent as a tool or MCP server, the wrapper must describe the agent's capability, side effects, timeout, cost envelope, and approval behavior. Do not hide a broad agent behind a narrow-looking tool name.

## Self-Improving Skills

Useful pattern: production runs generate candidate improvements. Dangerous pattern: the agent silently rewrites its own operating instructions.

Promotion pipeline:

```text
approved run
-> redacted trace
-> context card / lesson candidate
-> eval case
-> skill patch candidate
-> human or owner review
-> regression eval
-> versioned promotion
```

Hard rules:
- No raw PII or secrets in reusable skill/memory artifacts.
- No generated skill can modify permissions, auth, sandboxing, billing, retention, or connector scopes.
- Keep a decision log: why this skill exists, what failure it prevents, and what eval proves it.
- Remove or demote skills that cause false activation, stale advice, or duplicated context.

## Kill Criteria

Disable, rewrite, or quarantine a skill/connector when any of these are true:

- Its trigger overlaps another skill and no router/priority rule resolves the conflict.
- It only describes process and has no validation signal.
- It requires permission, auth, sandbox, connector scope, or billing changes in skill text.
- It injects large examples or references by default instead of lazy-loading them.
- It has no negative activation evals.
- It causes a measurable rise in false activation, token cost, or tool errors.
- It was generated from a run that was not reviewed and redacted.
- Its source path, owner, or version/hash cannot be traced from the run log.
