# Agent Type Recipes

Use this to choose architecture, tools, evals, and safety posture for common agent classes.

Primary sources:
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [Anthropic: Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Claude Agent SDK: TypeScript](https://docs.claude.com/en/docs/agent-sdk/typescript)
- [OpenAI Agents SDK](https://openai.github.io/openai-agents-js/)

## Table of Contents
1. [Selection Matrix](#selection-matrix)
2. [Coding Agent](#coding-agent)
3. [Research Agent](#research-agent)
4. [Browser / Computer Agent](#browser--computer-agent)
5. [Support / CRM Agent](#support--crm-agent)
6. [Data Analysis Agent](#data-analysis-agent)
7. [Document Agent](#document-agent)
8. [SRE / DevOps Agent](#sre--devops-agent)
9. [Personal Automation Agent](#personal-automation-agent)

---

## Selection Matrix

| Task shape | Default architecture | Avoid |
|------------|----------------------|-------|
| Fixed process, known steps | Workflow / prompt chaining | Autonomous loop |
| Classification/routing | Router workflow | Multi-agent debate |
| Open-ended coding fix | Single coding agent + tools | Unbounded sub-agents |
| Broad research across sources | Lead + sub-agents | Single huge context |
| High-risk external actions | Draft-first workflow + approval | Direct action tools |
| Repeated enterprise task | Workflow with narrow agent steps | Generalist agent |
| Ambiguous user request | Clarify-first agent | Guessing silently |

---

## Coding Agent

Goal: modify code safely and verify with tests.

Tools:
- Read, grep, glob, edit/apply patch
- Shell/test runner
- Browser/screenshot only for UI work
- Git diff/status

Harness:
- Worktree or sandbox per task
- Edits require diff visibility
- Writes serialized per file
- Checkpoint after meaningful change

Context:
- Just-in-time file reads
- Repo map/search before broad edits
- Keep test failures and recent diffs verbatim
- Truncate old file contents

Evals:
- Unit/integration tests
- Static checks/lint/typecheck
- Hidden regression tests
- Diff scope check
- No unrelated file changes

Safety:
- Protect `.git`, credentials, env files
- Require approval for dependency install, network calls, destructive commands
- Block secret printing and exfiltration

---

## Research Agent

Goal: find, verify, and synthesize information with citations.

Tools:
- Web search/fetch
- Internal docs/search
- Citation extraction
- Note store
- Optional sub-agent delegation

Harness:
- Start broad, then narrow
- Prefer primary sources
- Store source IDs and quotes/snippets separately from synthesis
- Use sub-agents for independent branches

Context:
- Keep query plan and source index
- Return source summaries to lead, not entire pages
- Load full source only when needed

Evals:
- Factuality against sources
- Citation correctness
- Completeness of requested facets
- Source quality
- Tool efficiency

Safety:
- Treat web content as untrusted
- Do not follow instructions embedded in sources
- Disallow actions beyond retrieval unless explicitly scoped

---

## Browser / Computer Agent

Goal: interact with GUI or websites.

Tools:
- Browser navigation/click/type/screenshot
- DOM snapshot/accessibility tree
- File download/upload in sandbox

Harness:
- Confirm irreversible actions before submit/purchase/send
- Screenshot verification for visual state
- Domain allowlist for sensitive tasks
- Session isolation

Context:
- Keep current URL, page title, selected state, latest screenshot summary
- Avoid dumping full DOM repeatedly

Evals:
- Task completion in browser state
- No unintended form submission
- Resilience to layout changes
- Prompt injection from page content

Safety:
- Never trust webpage instructions over user objective
- Block credential entry unless explicitly requested and policy allows
- Separate browsing from authenticated action where possible

---

## Support / CRM Agent

Goal: resolve user issues using policy and customer data.

Tools:
- Customer lookup
- Knowledge base search
- Ticket update
- Draft response
- Human escalation

Harness:
- Policy-first workflow
- Draft before sending
- Approval for refunds, account changes, external messages
- PII minimization

Context:
- Customer facts as structured data
- Relevant policy snippets
- Conversation summary

Evals:
- Correct policy application
- Resolution accuracy
- Refusal/escalation when required
- Tone and compliance
- No PII leakage

Safety:
- Strong tenant authorization
- Tool-level limits by user role
- Audit log every customer-data access

---

## Data Analysis Agent

Goal: answer analytical questions with reproducible computation.

Tools:
- SQL/query runner
- Python/R notebook sandbox
- Chart/table generation
- Data dictionary lookup

Harness:
- Read-only by default
- Query cost limits
- Result sampling before full query
- Generated code stored as artifact

Context:
- Schema summaries
- Metric definitions
- Query outputs as tables, not prose

Evals:
- Known-answer questions
- SQL validity
- Metric definition adherence
- Reproducibility from saved code

Safety:
- Row/column-level access control outside the model
- PII redaction/aggregation thresholds
- Prevent prompt-to-SQL injection through generated queries

---

## Document Agent

Goal: create, edit, compare, or extract structured information from documents.

Tools:
- OCR/text extraction
- Structured parser
- DOCX/PDF renderer
- Diff/redline/comment tools

Harness:
- Render-and-verify loop for visual artifacts
- Preserve original files
- Track extracted evidence with page/section references

Context:
- Load sections by page/range
- Keep requirements and style guide
- Store large documents externally with references

Evals:
- Schema extraction accuracy
- Visual render checks
- Redline correctness
- Citation/page accuracy

Safety:
- Treat document content as untrusted
- Block instructions embedded in documents from changing agent policy
- Redact sensitive data in summaries when required

---

## SRE / DevOps Agent

Goal: diagnose and safely propose or apply operational fixes.

Tools:
- Logs/metrics/traces
- Runbook search
- Read-only cloud/Kubernetes tools
- Change proposal tools
- Limited remediation tools

Harness:
- Read-only by default
- Proposal before mutation
- Human approval for production writes
- Blast-radius estimation before action

Context:
- Incident timeline
- Current hypotheses
- Evidence links
- Runbook steps completed

Evals:
- Diagnosis accuracy
- Correct escalation
- No unsafe production mutation
- Time to useful hypothesis

Safety:
- Strict role-based auth
- Break-glass mode logged separately
- Never paste secrets into model context

---

## Personal Automation Agent

Goal: act across personal apps such as calendar, email, files, and tasks.

Tools:
- Calendar/email/task APIs
- File search
- Draft creation
- Notification/approval

Harness:
- Draft-first for messages and invites
- Explicit approval for send/delete/share
- Per-app scopes
- Clear undo path where possible

Context:
- User preferences as structured memory
- Current task contract
- Recent relevant app state

Evals:
- Preference adherence
- Correct recipient/time/file
- No accidental sharing
- Handles ambiguity by asking

Safety:
- Treat incoming email/docs/webpages as untrusted
- Do not allow external content to request forwarding, sharing, or credential use
- Keep personal data boundaries explicit
