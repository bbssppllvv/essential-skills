# Reasoning Tokens Documentation

## Overview

OpenRouter's API supports **reasoning tokens** (also called thinking tokens) for compatible models. These tokens provide transparency into a model's step-by-step reasoning process and are charged as output tokens.

## Key Features

- Reasoning tokens appear in the `reasoning` field of responses by default
- Some models (like OpenAI's o-series) don't return reasoning tokens despite supporting them
- A unified `reasoning` parameter normalizes different provider implementations

## Controlling Reasoning Tokens

Use the `reasoning` parameter with these options:

**Effort-based control** (OpenAI, Grok models):
- `"xhigh"` (~95% of max_tokens)
- `"high"` (~80% of max_tokens)
- `"medium"` (~50% of max_tokens)
- `"low"` (~20% of max_tokens)
- `"minimal"` (~10% of max_tokens)
- `"none"` (disables reasoning)

**Token-based control** (Anthropic, Gemini, Qwen):
- `"max_tokens": 2000` specifies exact reasoning token allocation

**Other options**:
- `"exclude": true` — model reasons internally but doesn't return reasoning
- `"enabled": true` — enables reasoning at medium effort level

## Preserving Reasoning Across Turns

Pass reasoning back to the API using either:
1. `message.reasoning` (plaintext string)
2. `message.reasoning_details` (complete reasoning structure with metadata)

This is critical for tool use, as it maintains reasoning continuity when models pause to call external tools.

## Provider-Specific Implementation

**Anthropic models**: Use `reasoning.max_tokens` with minimum 1024 tokens and maximum 128,000 tokens. Formula: `budget_tokens = max(min(max_tokens * effort_ratio, 128000), 1024)`

**Google Gemini 3 models**: Maps `reasoning.effort` to Google's `thinkingLevel`. Note that "actual token consumption is determined internally by Google" rather than precise token control.

## Response Structure

Non-streaming responses include `reasoning_details` in `choices[].message`. Streaming responses include it in delta chunks. The `reasoning_details` array contains typed objects: `reasoning.summary`, `reasoning.encrypted`, or `reasoning.text`.

## Billing

Reasoning tokens count as output tokens for billing purposes, potentially increasing costs while improving response quality.
