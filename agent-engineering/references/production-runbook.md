# Production Runbook — Operating Agents

Use this when turning a prototype into a service or internal tool that people depend on.

Primary sources:
- [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [OpenAI: Agent evals](https://platform.openai.com/docs/guides/agent-evals)
- [OpenTelemetry](https://opentelemetry.io/)

## Table of Contents
1. [Production Readiness](#production-readiness)
2. [Agent-Legible Operating Environment](#agent-legible-operating-environment)
3. [Dry-Run Readiness Preview](#dry-run-readiness-preview)
4. [Observability](#observability)
5. [Reliability](#reliability)
6. [State and Persistence](#state-and-persistence)
7. [Cost Management](#cost-management)
8. [Deployment](#deployment)
9. [Incident Response](#incident-response)

---

## Production Readiness

Before production, define:
- Owner and on-call path.
- Supported tasks and explicit non-goals.
- Tool permission matrix.
- Eval suite and release gate.
- Source-of-truth artifacts and the tools that make them agent-legible.
- Validation signals the agent can inspect without human copy/paste.
- Data retention and privacy policy.
- Rollback path for prompt, tool, model, and harness changes.
- Kill switch for the agent and each high-risk tool.

Production agent contract:
```json
{
  "agent": "support-refund-agent",
  "supported_intents": ["refund_status", "refund_request_draft"],
  "forbidden_actions": ["issue_refund_without_approval"],
  "approval_required": ["refund_amount > 0", "external_message_send"],
  "max_cost_per_run_usd": 0.25,
  "max_wall_time_seconds": 120,
  "trace_retention_days": 30
}
```

---

## Agent-Legible Operating Environment

Before raising autonomy, make the environment inspectable:

- A short instruction map points to policies, runbooks, architecture docs, active plans, generated schemas, quality scorecards, and eval fixtures.
- The agent can query the same evidence humans use: app state, screenshots, logs, metrics, traces, tickets, ledgers, documents, or source indexes.
- Progress and approval state are durable artifacts, not only chat history.
- Repeated review comments become validators, lints, policy checks, or eval cases.
- Stale docs, unused tools, duplicate rules, and weak examples are removed on a recurring cadence.

This is a launch requirement, not polish. If the agent cannot retrieve the rule or validate the outcome, adding autonomy mostly increases the blast radius.

---

## Dry-Run Readiness Preview

Before a live run in a complex harness, provide a preview path that resolves configuration without executing the model, tools, MCP calls, or sub-agents.

A useful dry run reports:
- Resolved model/provider settings and missing auth.
- Prompt bundle, active instruction files, matching skills, and source paths.
- Tool set, MCP/connectors, resource roots, and obvious broken server config.
- Permission mode, high-risk tools, sandbox status, and approval policy.
- Cost/time/turn budgets and whether the run is likely read-only or stateful.
- Verdict: `ready`, `warning`, or `blocked`.
- Concrete `next_actions`.

Use dry run for onboarding, incident triage, untrusted repositories, new MCP servers, and launch checks. It catches missing credentials, stale skill paths, broken connector config, unsafe project skills, absent sandbox runtime, and permission surprises before the agent consumes tokens or mutates state.

---

## Observability

Log a trace tree, not flat strings.

Trace hierarchy:
```text
user_request
  agent_run
    model_step
      tool_call
      tool_result
    sub_agent_run
      model_step
      tool_call
    final_response
    graders
```

Required fields:
- `trace_id`, `session_id`, `user_id` or tenant-safe pseudonym
- agent version, prompt version, tool schema version
- model provider, model ID, temperature/reasoning settings
- input/output token usage, cache reads/writes, cost estimate
- tool name, args hash, redacted args, status, duration, side-effect flag
- permission decision and reason
- context compaction events
- errors, retries, denials, user approvals
- final artifact pointers

Never rely only on provider dashboards. You need app-layer traces that connect LLM calls to downstream effects.

---

## Reliability

Controls:
- Per-model-call timeout.
- Per-tool timeout.
- Max tool calls per turn.
- Max turns per run.
- Max total cost per run.
- Retry only idempotent or explicitly safe operations.
- Respect `retry-after` for rate limits.
- Abort/cancel support.
- Stuck-loop detection using repeated tool signatures and no-progress state.

Tool execution:
- Make side-effecting tools idempotent with request IDs.
- Prefer draft/preview tools before commit tools.
- Use compensation actions when rollback is possible.
- Serialize writes to the same resource.
- Snapshot state before risky edits.

Failure response:
- State what completed.
- State what did not complete.
- Preserve artifacts for continuation.
- Offer a concrete next action.

---

## State and Persistence

Persist state outside the model:
- Append-only transcript for audit and replay.
- Current task plan.
- Completed steps.
- Pending approvals.
- Artifact locations.
- Tool-specific state.
- Evaluation outcomes.

For long-running agents:
- Start every session by reading durable progress state.
- Store progress in structured JSON where possible.
- Commit or checkpoint after meaningful progress.
- Resume from checkpoints, not from memory.
- Do not restore elevated permissions silently on resume.

For multi-tenant or concurrent MCP/server deployments:
- Scope logs, notifications, temporary tool results, resource subscriptions, and auth state to the request/session.
- Test that one client cannot receive another client's logs or notifications.
- Require explicit `thread_id`/session IDs for checkpointed workflows.
- Validate checkpoint serialization settings and retention before enabling persistence.

State file example:
```json
{
  "task_id": "migration-042",
  "objective": "Migrate billing API to v2",
  "completed": ["schema updated", "client generated"],
  "current_blocker": "contract tests failing on invoice totals",
  "next_step": "inspect failing contract test output",
  "modified_files": ["src/billing/client.ts"],
  "pending_approval": null
}
```

---

## Cost Management

Track cost per successful task, not just per request.

Controls:
- Route simple steps to cheaper models.
- Cache stable prompts/tool schemas/static context.
- Mask/truncate old observations before summarizing.
- Disable sub-agents by default for simple tasks.
- Cap fan-out and tool calls.
- Use batch jobs for offline workloads.
- Alert on cost/run, cost/user/day, and sudden cache miss rate changes.

Cost review questions:
- Did the agent use the right model for the step?
- Did context include irrelevant tool output?
- Did failed tool calls cause repeated model turns?
- Did sub-agents duplicate work?
- Did the agent continue after success criteria were already met?

---

## Deployment

Version all agent surfaces:
- System prompt
- Tool descriptions and schemas
- Tool implementations
- Permission policy
- Model IDs and settings
- Context compaction rules
- Eval dataset

Deployment pattern:
1. Run offline eval suite.
2. Run security/adversarial suite.
3. Shadow traffic where possible.
4. Canary to small user group.
5. Monitor success, safety, latency, cost, and escalations.
6. Roll forward or rollback.

Model upgrades require the same process as code changes. Never swap a model alias in production without evals.

---

## Incident Response

Incident triggers:
- Unauthorized tool action.
- Secret exposure.
- Excessive spend or runaway loop.
- Bad external communication.
- Data integrity issue.
- Repeated customer-facing failure.

Immediate actions:
1. Disable affected tool or agent route.
2. Revoke or rotate credentials if exposed.
3. Preserve traces and artifacts.
4. Identify prompt/tool/model/policy version.
5. Replay in isolated environment.
6. Add regression eval before fixing.
7. Patch harness/policy before prompt-only mitigation when possible.

Postmortem should answer:
- What boundary failed?
- Was the model allowed to request too much?
- Did the harness enforce policy?
- Did evals cover this class?
- Did observability show the failure early enough?
- What single release gate would have caught it?
