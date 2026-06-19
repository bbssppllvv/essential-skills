---
name: crucible
description: >-
  Run an independent multi-perspective review by spawning a small panel of
  subagents, each with a sharp role, then synthesizing their findings into
  evidence-based decisions. Invoke autonomously only when independent judgment
  is likely to materially change a decision before action: meaningful risk, real
  uncertainty, competing approaches, a stuck diagnosis, a plan that should be
  stress-tested, or an explicit request for fresh eyes, multiple reviewers, a
  devil's advocate, red team, jury, outside opinion, or second opinion. Do not
  invoke for routine low-risk edits, direct factual answers or commands,
  mechanical docs/tests, tasks without a reviewable artifact, or when already
  acting as a Crucible reviewer/subagent.
---

# Crucible

A crucible is a deliberately sharp review panel. Use it to expose blind spots before shipping code, committing to a plan, or making a high-impact recommendation.

Core rule: subagents are advisors, not authorities. Verify every claim against source artifacts before presenting it or acting on it.

Recursion guard: when acting as a Crucible reviewer or subagent, do not invoke Crucible or spawn additional reviewers. Run at most one Crucible panel per top-level task unless the user explicitly asks for nested review.

## Operating Model

- The main agent is the planner, integrator, and decision-maker.
- Reviewers are sidecars. They do not coordinate with each other, negotiate scope, or create new requirements.
- Keep the critical path local. Do not delegate the decision itself, integration, or work whose result is needed before the next immediate local step.
- Spawn only concrete, bounded review tasks that can run independently and materially improve the outcome.
- Stop spawning once the remaining work is sequential, small, or integration-heavy. Finish locally.

## Invocation Model

Use this skill based on expected value, not task labels. The question is:

> Would a few independent, role-focused reviewers likely find a real issue, missing assumption, better framing, or sharper tradeoff before I act?

If yes, use Crucible without waiting for the user to ask. Invoke it after enough inspection to provide reviewers with concrete artifacts, and before the main agent finalizes a plan, applies risky edits, presents a review, or makes a recommendation.

Strong signals:

- Consequence: a wrong answer could cause a regression, bad architecture, user harm, security/privacy risk, wasted work, data loss, or a misleading recommendation.
- Uncertainty: there are multiple plausible solutions, incomplete context, hidden assumptions, or weak confidence.
- Perspective gap: the work spans concerns one linear pass may underweight, such as correctness, simplicity, tests, product value, platform conventions, or operations.
- Commitment point: the main agent has drafted a risky or uncertain plan, review, diagnosis, or implementation and is about to act on it.
- Stuckness: debugging or planning has started to orbit one hypothesis.
- User intent: the user asks for outside opinions, fresh eyes, multiple agents, review, validation, critique, red-team thinking, or a devil's advocate.

Do not treat this as a rigid checklist. One strong signal can be enough when it implies material expected value; several weak signals can also be enough. Skip when the likely benefit is low: the task is local and obvious, the answer is a deterministic command/fact lookup, the edit is mechanical docs or low-risk test cleanup, there is no meaningful artifact to review, the added latency would dominate the work, or the review would require sharing irrelevant private context.

## Panel Size

- 1 reviewer: quick sanity check.
- 2 reviewers: focused disagreement; choose opposing roles.
- 3 reviewers: default for PR review, plan review, and architecture critique.
- 4-5 reviewers: broad review for high-risk or cross-cutting work.
- 6+ reviewers: avoid unless the user explicitly asks for a large panel.

Pick the smallest panel that covers the real risk. Role diversity matters more than headcount.

## Role Catalog

Choose roles that create useful tension:

| Role | Use When |
| --- | --- |
| Devil's Advocate | A plan needs to be broken before implementation. |
| Pragmatist | The solution may be overbuilt, slow, or too clever. |
| Correctness Hunter | Code may have edge-case bugs, broken invariants, or bad assumptions. |
| Regression Hunter | Existing behavior must not be disturbed. |
| Security/Privacy Reviewer | Secrets, permissions, automation, user data, or network calls are involved. |
| Test/QA Reviewer | Coverage, manual QA, flaky assumptions, or release confidence matter. |
| Maintainability Reviewer | Ownership, boundaries, naming, readability, or future changes are in play. |
| Performance/Reliability Reviewer | Latency, memory, concurrency, retries, rate limits, or failure modes matter. |
| Platform Reviewer | Native platform conventions, lifecycle, entitlements, or APIs matter. |
| Product/User Advocate | The work may miss the user's actual job, trust, or workflow. |
| UX/Accessibility Reviewer | Screens, flows, interaction states, copy, or accessibility are affected. |

Default panels:

- PR review: Correctness Hunter, Regression Hunter, Test/QA Reviewer.
- Risky PR: Correctness Hunter, Security/Privacy Reviewer, Regression Hunter, Pragmatist.
- Plan review: Devil's Advocate, Pragmatist, Risk/QA perspective via Test/QA Reviewer.
- Architecture review: Maintainability Reviewer, Performance/Reliability Reviewer, Pragmatist.
- Product/UX review: Product/User Advocate, UX/Accessibility Reviewer, Pragmatist.

## Workflow

1. Inspect just enough context to understand the artifact and the stakes.
2. Decide whether independent perspectives are likely to change the answer, plan, review, or implementation.
3. Define the artifact under review: diff, files, plan, logs, screenshot, design, diagnosis, or proposal.
4. Minimize context before delegation: prefer diffs, excerpts, file paths, screenshots, or logs with secrets and irrelevant private content removed. State what reviewers may inspect and what was withheld.
5. Select 1-5 roles based on the risks and unknowns that matter most.
6. Spawn independent subagents. If subagent tools are not already available, make one bounded search for multi-agent tools.
7. If true subagents are unavailable, do not claim independent review happened. Either proceed with a clearly labeled single-agent structured review, or report the blocker if the user explicitly required spawned agents.
8. Give each subagent the same minimized artifact package and one role-specific mandate. Do not give subagents each other's outputs.
9. Require structured findings with evidence, confidence, and scope boundaries.
10. Synthesize after reviewers return. If some agents fail or time out, synthesize only available outputs and label the result partial. Never invent missing reviewer responses.
11. Act on the synthesis: revise the plan, implement changes, or present final review findings.

## Subagent Prompt

Use this shape and fill only the task-specific blanks:

```text
You are the [ROLE] in a crucible review.

Task: [what is being reviewed and why]
Artifact: [diff/files/plan/logs/screenshot/etc.]
Allowed inspection scope: [whether the reviewer may inspect files beyond the artifact]

Review only from your assigned perspective. Be sharp but evidence-based.
Do not invoke Crucible, spawn subagents, or perform a second-level panel review.
Do not expand the project scope. Do not propose optional enhancements, rewrites, redesigns, speculative future work, or nice-to-haves unless they materially affect the stated goal.

Return:
- Findings: severity, evidence, confidence, and recommended action.
- Questions or missing context that would change your conclusion.
- Scope you intentionally did not review.

Use severity Critical/High/Medium/Low. Evidence must be a file:line reference, exact artifact excerpt, command output, screenshot region, or "not evidenced." Confidence must be High/Medium/Low.
Make the handoff usable without reading your full transcript.

Prefer fewer high-signal findings over broad commentary. Skip nits unless they change correctness, user trust, maintainability, verification confidence, or delivery risk. Do not assume facts not present in the artifact.
```

## Synthesis Rules

- Treat repeated findings as stronger only after verifying the underlying evidence.
- Keep good disagreement visible; do not average it away.
- Reject findings that are vague, unsupported, out of scope, or based on invented context.
- Track each material finding as accepted, rejected, or unresolved, with a short reason.
- Accept feedback only when it is inside the original goal and materially improves correctness, risk, user value, maintainability, or verification confidence.
- Convert accepted feedback into concrete actions: code changes, plan edits, tests, QA notes, or explicit non-actions.
- Park useful but nonessential ideas as follow-up notes; do not silently expand the current task to include them.
- Do not run another panel just to arbitrate minor disagreements. The main agent decides.
- For code review, lead with actionable findings ordered by severity and include file/line references when available.
- For plan review, return the revised plan plus the specific risks or assumptions that changed.
- If the panel finds no material issue, say so and name the residual risk or test gap.

## Pitfalls

- Do not use a big panel to compensate for unclear goals. Clarify the goal or inspect the artifact first.
- Do not let subagents vote on truth. Evidence beats consensus.
- Do not spawn multiple agents with the same role unless the artifact is large enough to shard by area.
- Do not leak secrets, credentials, private user content, or unrelated repository context into subagent prompts.
- Do not let reviewer imagination turn into scope creep. Roles are lenses, not product requirements.
- Do not let the review become theater. A useful crucible produces decisions, not just opinions.
