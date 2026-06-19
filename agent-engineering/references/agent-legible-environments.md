# Agent-Legible Environments

Use this when a harness keeps failing because the agent lacks the right source of truth, cannot validate its own work, or depends on humans to paste context back into chat.

Primary sources:
- [OpenAI: Harness Engineering](https://openai.com/index/harness-engineering/)
- [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic: Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)

## Core Rule

An agent can only use what the harness makes legible: retrievable source-of-truth artifacts, current workflow state, validation signals, and audit evidence. Knowledge that lives only in chat, meetings, private docs, or human memory should be treated as operationally absent.

The top-level instruction file should be a map, not an encyclopedia. Keep it short and stable, then point to structured references loaded just in time.

## What To Expose

Expose domain facts through approved tools or versioned artifacts:

```text
product/coding: architecture docs, plans, test commands, app screenshots, logs, metrics, traces
support: policy docs, ticket history, customer state, escalation rules, approved response examples
finance: ledger views, reconciliation checks, approval policy, audit events
legal: clause library, contract corpus, redline history, jurisdiction rules, review rubric
research: source corpus, extraction rubric, citation checks, screening decisions
ops: runbooks, service topology, incident timeline, logs, metrics, rollback policy
```

For each domain, define the signals that prove progress. Good signals are directly checkable: tests pass, metric recovered, policy citation present, PII redacted, approval attached, screenshot changed, audit event written, or source table completed.

## Source-Of-Truth Layout

A durable knowledge base should include:

```text
agent-instructions.md       # short map, operating rules, where to look next
architecture.md             # domain model and boundaries
policies/                   # authority, compliance, escalation, safety
runbooks/                   # repeatable procedures
plans/active/               # current plans and progress logs
plans/completed/            # completed decisions and tradeoffs
generated/                  # schema inventories, API catalogs, indexes
quality/                    # scorecards, known gaps, freshness status
evals/                      # fixtures and regression tasks
```

Each long-lived artifact should carry minimal metadata: owner, scope, last reviewed date, source of truth, verification status, and known staleness risks.

## Mechanical Invariants

Do not rely on prompt advice for recurring correctness rules. Promote repeated review comments into checks:

```text
schema validators
permission and approval gates
policy checkers
source-citation validators
PII/secret scanners
dependency-boundary lints
UI/accessibility checks
freshness checks
cost and latency budgets
regression evals
```

Validators should return remediation messages that are safe to show the model:

```json
{
  "status": "error",
  "type": "approval_required",
  "message": "External email send requires an approval record.",
  "next_valid_actions": ["draft_email", "request_approval"]
}
```

## Feedback Loop

When the agent fails, do not only rewrite the prompt. Identify the missing harness component:

```text
missing source of truth
missing tool or connector
missing validator
missing permission rule
missing sandbox signal
missing eval fixture
missing recovery path
missing progress artifact
```

Then encode the fix into docs, tools, schemas, policies, evals, or CI so the improvement compounds.

## Long-Running Work

For work that spans context windows, keep progress outside the prompt:

```text
current objective
active plan
latest checkpoint
evidence gathered
files/artifacts changed
approval state
open blockers
next action
do-not-redo list
```

Anthropic's long-running harness writeup uses a progress file, structured feature/test list, init script, and git history so a fresh agent can regain state quickly. Generalize that pattern to the domain rather than relying on conversational memory.

## Entropy Management

Agents imitate the environment. If weak examples, stale docs, obsolete tools, and duplicated rules accumulate, future runs reproduce them.

Run recurring cleanup jobs:

```text
stale documentation scans
tool inventory review
duplicate instruction merge
quality score updates
known-gap review
stale plan archival
repeated-failure clustering
prompt/tool bundle review
eval additions after incidents
```

Keep the golden rules few, visible, and mechanically enforced where possible.

## Anti-Patterns

- One giant instruction file that competes with task context.
- Validation signals that exist only in a human's browser, dashboard, or memory.
- Review comments repeated in prose but never converted into checks.
- Approval state stored only in chat summaries.
- External documents treated as authoritative instructions.
- Stale examples left in place because "the model should know better."
- Autonomy increased before the environment exposes enough evidence for self-verification.
