# Guardrails

Control spending and model access for your organization.

Set spending limits, restrict model access, and enforce data policies for your organization members and API keys with OpenRouter guardrails.

## Overview

Guardrails let organizations control how their members and API keys can use OpenRouter. Organizations can establish spending limits, restrict available models and providers, and enforce data privacy policies.

Account-wide settings remain in effect. Guardrails enable stricter controls for individual API keys or users.

## Enabling Guardrails

To set up guardrails:

1. Navigate to **Settings > Privacy** in your OpenRouter dashboard
2. Scroll to the **Guardrails** section
3. Click **"New Guardrail"** to create your first guardrail

> **Tip:** Organization administrators must create and manage guardrails.

## Guardrail Settings

Each guardrail can include any combination of:

- **Budget limit** — Spending cap in USD that resets daily, weekly, or monthly. Requests are rejected when the limit is reached.
- **Model allowlist** — Restrict to specific models. Leave empty to allow all.
- **Provider allowlist** — Restrict to specific providers. Leave empty to allow all.
- **Zero Data Retention** — Require ZDR-compatible providers for all requests.

> **Note:** Individual API key budgets still apply. The lower limit wins.

## Assigning Guardrails

- **Member assignments** — Assign to specific organization members. Sets a baseline for all their API keys and chatroom usage.
- **API key assignments** — Assign directly to specific keys for granular control. Layers on top of member guardrails.

Only one guardrail can be directly assigned to a user or key. All API keys created by organization members implicitly follow that member's guardrail assignment, even if the API Key has further restrictions.

## Guardrail Hierarchy

Account-wide privacy and provider settings are enforced as a default guardrail. When additional guardrails apply, they combine using:

- **Provider allowlists:** Intersection across all guardrails
- **Model allowlists:** Intersection across all guardrails
- **Zero Data Retention:** OR logic
- **Budget limits:** Each guardrail's budget is checked independently

Stricter rules always win when multiple guardrails apply.

## Eligibility Preview

When viewing a guardrail, an eligibility preview shows which providers and models are available with that guardrail combined with account settings.

## Budget Enforcement

Guardrail budgets are enforced per-user and per-key, not shared across all users with that guardrail.

**Example 1:** A $50/day member guardrail assigned to three team members provides each with their own $50/day allowance.

**Example 2:** If Alice creates two API keys with $20/day limits each (spending $15 and $10 respectively), her total usage is $25. A member guardrail with a $20/day limit would block her requests.

**Example 3:** A member guardrail of $100/day with an API key guardrail of $30/day means the key can spend $30/day, while total usage across all keys cannot exceed $100/day.

## API Access

You can manage guardrails programmatically using the OpenRouter API. Create, update, delete, and assign guardrails to API keys and organization members directly from code.

See the [Guardrails API reference](https://openrouter.ai/docs/api/api-reference/guardrails/list-guardrails) for available endpoints and usage examples.
