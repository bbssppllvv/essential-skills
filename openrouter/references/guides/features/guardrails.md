# Guardrails Documentation

## Overview

OpenRouter's guardrails feature enables organizations to govern member and API key usage. According to the documentation, these controls help "set spending limits, restrict which models and providers are available, and enforce data privacy policies."

## Core Features

Guardrails support four primary control mechanisms:

1. **Budget limits** - Daily, weekly, or monthly spending caps in USD with automatic request rejection upon limit breach
2. **Model allowlist** - Restriction to specified models (empty = all allowed)
3. **Provider allowlist** - Restriction to specified providers (empty = all allowed)
4. **Zero Data Retention** - Enforcement of ZDR-compatible providers

## Assignment Levels

Guardrails can be applied at two tiers:
- **Member level** - Establishes baseline restrictions for all user-created API keys
- **API key level** - Provides granular control layered atop member restrictions

## Rule Combination Logic

When multiple guardrails apply, the documentation specifies these enforcement rules:

- Provider and model allowlists use intersection logic (most restrictive wins)
- Zero Data Retention uses OR logic (enforced if any guardrail requires it)
- Budget limits are evaluated independently per user and per key

## Budget Mechanics

The system tracks spending separately: "each member gets their own allowance" and "API key usage accumulates to member usage." Multiple keys from one user compound toward the member's total budget.

## Setup and API Access

Configuration occurs via Settings > Privacy in the dashboard. Organization administrators manage guardrails programmatically through dedicated API endpoints.
