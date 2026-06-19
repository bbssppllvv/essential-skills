# Evaluation Harness — Production Playbook

Use this when designing, shipping, or changing an agent. Evals are not a separate QA phase; they are the control system for agent development.

Primary sources:
- [Anthropic: Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [OpenAI: Agent evals](https://platform.openai.com/docs/guides/agent-evals)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)

## Table of Contents
1. [What to Measure](#what-to-measure)
2. [Dataset Design](#dataset-design)
3. [Graders](#graders)
4. [Metrics](#metrics)
5. [Harness Architecture](#harness-architecture)
6. [Skill and Harness Feature Evals](#skill-and-harness-feature-evals)
7. [Production Trace Flywheel](#production-trace-flywheel)
8. [Release Gates](#release-gates)
9. [Common Eval Bugs](#common-eval-bugs)

---

## What to Measure

Measure outcomes, not exact paths. A good eval asks: did the agent complete the user goal within the allowed risk, cost, and time budget?

Core dimensions:
- **Task success**: final state satisfies user intent.
- **Safety**: no unauthorized tool use, data leak, destructive action, or policy violation.
- **Robustness**: succeeds across retries, paraphrases, noisy inputs, and tool failures.
- **Efficiency**: tokens, wall time, tool calls, retries, and unnecessary delegation.
- **User experience**: progress visibility, useful clarification, clean failure messages.

For agents that change the world, grade the environment state, not the chat text. Examples: tests pass, database row updated correctly, ticket has the right status, email draft is safe and unsent until approved.

---

## Dataset Design

Start with **20-50 tasks from real failures or manual checks**. Add more only when the agent matures enough that small regressions need finer statistical power.

Each task should include:

```json
{
  "id": "auth-refresh-001",
  "category": "coding.fix",
  "input": "Fix refresh token expiry handling.",
  "environment": {
    "repo": "fixtures/auth-service",
    "setup": "npm ci",
    "reset": "git clean -fdx && git checkout -- ."
  },
  "allowed_tools": ["read_file", "edit_file", "bash"],
  "success_criteria": [
    "npm test passes",
    "refresh tokens expire according to config",
    "no unrelated files modified"
  ],
  "negative_criteria": [
    "does not weaken auth checks",
    "does not log tokens"
  ],
  "reference_solution": "patches/auth-refresh-001.diff"
}
```

Dataset balance:
- Positive and negative cases: when the agent should act and when it should refuse or ask.
- Easy, medium, hard, adversarial.
- Tool failure cases: missing files, flaky API, invalid credentials, timeouts.
- Cost-sensitive cases: simple tasks that should not spawn sub-agents or use expensive models.

Task quality rule: two domain experts should independently reach the same pass/fail verdict. If they cannot, the task or grader is ambiguous.

### "Cursor Blame" — sourcing tasks from production

The highest-quality eval tasks come from real production usage, not hand-crafted scenarios. Cursor's approach: trace committed code back to the agent request that produced it ("Cursor Blame"). This creates natural query → ground-truth pairs from actual developer sessions.

Key properties of production-sourced tasks:
- **Brief descriptions** — real developers give short requests, not detailed GitHub issues. Eval tasks should match this.
- **Calibrated complexity** — Cursor's benchmark complexity doubled between versions as they tracked real usage evolution. Refresh the dataset periodically (Cursor: every few months).
- **Multiple valid solutions** — production tasks rarely have a single correct answer. Sourcing from real sessions naturally captures this; graders must handle it (see Agentic Graders below).

Source: [Cursor: How we compare model quality in Cursor (Mar 2026)](https://cursor.com/blog/cursorbench)

---

## Graders

Use the cheapest grader that reliably captures the requirement.

### Deterministic Graders

Best for code, data, schemas, permissions, and exact outcomes.

Examples:
- Unit/integration tests
- Snapshot diffs
- JSON schema validation
- SQL assertions
- File allowlist checks
- "No secrets in output" regex plus high-entropy scanner
- Tool-call policy replay

### Model Graders

Use for semantic quality, support tone, research completeness, and synthesis. Calibrate against human labels before trusting scores.

Rubric pattern:
```json
{
  "score": 0.0,
  "passed": false,
  "reasons": ["Specific mismatch"],
  "evidence": ["Quote or artifact ID"],
  "confidence": 0.0
}
```

Require evidence. A model grader without cited evidence is another unobserved agent.

### Agentic Graders

Standard model graders score against a reference solution — but many coding and agent tasks admit multiple valid approaches. An agentic grader runs the solution in the actual environment (executes tests, inspects files, queries the system) and grades the outcome rather than comparing to a canonical answer.

Use agentic graders when:
- Multiple correct implementations exist (algorithms, refactors, architecture decisions)
- The pass criterion is behavioral (tests pass, API returns correct response, UI renders correctly)
- String comparison would penalize valid alternative solutions

Tradeoff: agentic graders are slower and more expensive than model graders, but catch the failure mode where a solution looks correct to a text comparison but breaks in practice.

Source: [Cursor: How we compare model quality in Cursor (Mar 2026)](https://cursor.com/blog/cursorbench)

### Human Review

Use human review for:
- New task categories
- Safety boundary changes
- Customer-facing launch gates
- Disputes between deterministic and model graders
- Sampling production traces

---

## Metrics

Report more than one number.

- **pass@1**: first-try success. Best default for customer-facing agents.
- **pass@k**: probability at least one of k trials succeeds. Useful for proposal generation or offline workflows where retries are acceptable.
- **pass^k**: probability all k trials succeed. Useful for reliability expectations.
- **Safety pass rate**: no forbidden action across all trials.
- **Cost per successful task**: total cost / passed tasks, not average request cost.
- **Latency percentiles**: p50/p95/p99, plus "time to first useful progress".
- **Tool efficiency**: tool calls per task, duplicate calls, failed calls, unnecessary sub-agents.

### Production quality metrics (from Cursor, Apr 2026)

Two signals that capture real-world quality that benchmarks miss:

**Keep Rate** — for agent-generated artifacts (code, files, data), track what fraction remains unmodified after fixed time intervals (e.g., 5 min, 30 min, 1 hour). Low Keep Rate means users had to manually fix the agent's output or ask it to redo work. This is a strong lagging indicator of agent quality in production, requires no labeling, and degrades gracefully as usage grows.

**LLM semantic satisfaction scoring** — a small model reads the user's follow-up message after the agent responds and classifies satisfaction:
- Positive signals: moving to the next feature, confirming it worked, closing the session
- Negative signals: pasting a stack trace, asking the agent to redo, expressing frustration

```typescript
async function scoreUserSatisfaction(agentResponse: string, userFollowUp: string): Promise<"satisfied" | "unsatisfied" | "neutral"> {
  const result = await classify({
    model: "claude-haiku-4-5-20251001", // cheap — runs on every session
    prompt: `Agent responded with a code change. User followed up with:\n\n"${userFollowUp}"\n\nClassify: did the user accept the result (satisfied), need to fix/redo (unsatisfied), or continue unrelated (neutral)?`,
  });
  return result;
}
```

These two metrics complement offline benchmarks: benchmarks are fast and standardized; Keep Rate and satisfaction scoring catch real failures that synthetic tasks miss.

For model or harness changes, compare:
- old model + old harness
- new model + old harness
- old model + new harness
- new model + new harness

This separates model regression from harness regression.

---

## Harness Architecture

Minimum eval runner:

1. Reset environment from a clean fixture.
2. Start a new session with fixed config.
3. Run N independent trials per task.
4. Capture full trace: prompt, model, system prompt version, tool schemas, tool calls, tool outputs, artifacts, token usage, cost, wall time.
5. Run graders against artifacts and trace.
6. Store results in append-only JSONL.
7. Produce aggregate report and failure clusters.

Trace schema:
```json
{
  "run_id": "uuid",
  "task_id": "auth-refresh-001",
  "trial": 1,
  "agent_version": "git-sha",
  "model": "provider/model-id",
  "status": "pass|fail|error|blocked",
  "scores": { "task": 1, "safety": 1, "efficiency": 0.7 },
  "usage": { "input_tokens": 0, "output_tokens": 0, "cost_usd": 0 },
  "tool_summary": { "calls": 0, "failed": 0, "denied": 0 },
  "artifacts": ["path/to/trace.jsonl", "path/to/diff.patch"]
}
```

Every eval should be reproducible enough to debug:
- Pin package versions and model aliases where possible.
- Store exact prompts and tool schemas.
- Snapshot environment fixtures.
- Keep raw traces even when summaries exist.

---

## Skill and Harness Feature Evals

Skills, plugins, connectors, and harness middleware need their own evals. They are not safe just because they are "only instructions" or "just a wrapper."

### Skill activation evals

For every production skill, maintain:
- **Positive triggers**: realistic prompts that should load the skill.
- **Negative triggers**: adjacent prompts that should not load it.
- **Conflict triggers**: prompts where another skill is more appropriate.
- **Output assertions**: patterns the skill should cause and patterns it must not cause.
- **Loader checks**: frontmatter parses, referenced files exist, scripts are executable, and duplicate triggers are detected.

Acceptance criteria should be explicit enough that a future agent can improve the skill without changing the target. A useful skill eval often has the shape: "given prompt X, this skill should activate, load only these references, produce artifact Y, and avoid behavior Z."

When measuring whether a skill actually helps, compare it against a baseline:
- New skill: same prompt with the skill vs. without the skill.
- Skill improvement: same prompt with the new skill vs. a snapshot of the old skill.
- Run paired comparisons in the same batch when possible, so provider latency/model drift affects both sides similarly.
- Capture timing, tokens, output artifacts, and assertion results for both sides.
- Remove non-discriminating assertions that pass regardless of whether the skill was active.

Static skill checks are still useful as a cheap gate: description quality, trigger clarity, line count, reference existence, duplicate content, orphan references, bloated bodies, and missing examples. They do not replace run-based evals.

### Harness feature evals

When validating tool loops, permissions, compaction, sandboxing, skills, MCP, or sub-agents:
- Test on an unfamiliar workspace, not only the harness repository.
- Use real provider calls for API serialization, streaming, tool-calling, and SDK compatibility checks. Mocks are fine for unit tests but do not prove the agent loop.
- Use at least two turns when testing memory, compaction, skills, or recovery.
- Verify the tool trace and artifacts, not just final text.
- Combine features that interact in production, such as hooks + skills + tool execution or MCP + permissions + streaming.
- Classify failures before changing code: setup missing, timeout too low, max-turn cap too low, provider error, tool error with recovery, or product bug.

For harness optimization, separate train and holdout tasks. Let experiments edit prompts, tool schemas, skills, middleware, and registration only in a proposal branch/workspace. Promote the change only when holdout success improves without safety, latency, or cost regression.

---

## Production Trace Flywheel

Production traces are the best source of future evals, but raw traces are not reusable knowledge by default.

Promotion pipeline:

```text
production run
-> owner-approved run
-> redacted trace
-> failure cluster / lesson
-> eval case
-> context card or skill patch candidate
-> regression eval
-> versioned promotion
```

Rules:
- Do not train, fine-tune, or write reusable skills from raw production data.
- Require human or owner approval before promoting a run.
- Redact secrets, PII, tenant data, auth tokens, and private file contents before reuse.
- Store whether raw input was retained, redaction status, PII level, reviewer, and promotion target.
- Prefer turning failures into eval cases before turning them into new instructions.
- Track stale lessons and remove them when they no longer match current tools or product behavior.

Trace-derived evals should preserve the short, messy shape of real user prompts. Do not rewrite them into idealized tickets unless that is the product's actual input format.

---

## Release Gates

Suggested gates:

- **Prototype**: 20-50 tasks; read every failing transcript.
- **Internal beta**: regression suite passes; no critical safety failures; cost budget known.
- **External beta**: adversarial suite included; production tracing enabled; rollback plan tested.
- **General availability**: stable pass@1/pass^k trend; incident process; model upgrade protocol.

Block release when:
- Safety regression appears, even if task success improves.
- Cost per success rises without product justification.
- Model graders disagree with deterministic checks and no human adjudication exists.
- Failures cluster around ambiguous instructions or broken eval tasks.

---

## Common Eval Bugs

- Testing only happy paths.
- Grading exact tool sequence instead of final outcome.
- Missing reference solution, so a task may be impossible.
- Hidden assumptions in tests not visible in the task prompt.
- Model judge accepts fluent hallucinations.
- Using pass@k to claim production reliability.
- Comparing models without controlling harness changes.
- Not replaying traces after prompt/tool-schema changes.
- Ignoring tasks with 0% pass rate; often the task or grader is broken.

---

## Infrastructure Noise

Sandbox resource configuration is a hidden confounder in agentic evals. Anthropic quantified this on Terminal-Bench 2.0 (Feb 2026):

- **6 percentage point gap** between most and least resourced configurations (p < 0.01)
- Strict enforcement (container killed at spec limit): **5.8% infrastructure error rate**
- 3x headroom: **2.1% errors** — a statistically significant improvement
- Uncapped: **0.5% errors**

**The 3x rule**: below ~3x the per-task resource spec, additional resources fix reliability issues without changing what the benchmark measures. Above 3x, resources actively enable different problem-solving strategies (expensive dependencies, memory-intensive processes) — you're now measuring something different.

**Practical rules:**
1. Set guaranteed allocation and hard limit *separately* — identical values cause spurious OOM kills from transient memory spikes.
2. Set the ceiling at ~3x per-task spec to neutralize noise while maintaining meaningful resource pressure.
3. **Treat leaderboard gaps below 3 percentage points with skepticism** unless the infrastructure configuration (hardware, resource limits, enforcement method, concurrency) is documented and matched.
4. Run evaluations across multiple times and days to average out API latency and cluster health variance.

Infrastructure noise stacks *on top of* statistical confidence intervals, not within them — small gaps are unreliable without matched configs.

Source: [Anthropic: Quantifying infrastructure noise in agentic coding evals (Feb 2026)](https://www.anthropic.com/engineering/infrastructure-noise)
