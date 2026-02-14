# Usage Accounting Documentation

## Overview

OpenRouter's Usage Accounting feature enables developers to "track AI model usage without making additional API calls." The system automatically returns detailed token counts, costs, and caching information with every API response.

## Key Features

**Automatic Information Tracking:**
The API provides prompt tokens, completion tokens, reasoning tokens, and cached token counts using each model's native tokenizer. No extra parameters are required—this data is included in all responses by default.

**Response Structure:**
Every response contains a `usage` object with:
- `completion_tokens` and `prompt_tokens` counts
- `cost` in credits
- `cached_tokens` (tokens read from cache)
- `cache_write_tokens` (tokens written to cache)
- `total_tokens` sum

**Cost Details:**
The response includes `cost` (total charged) and `upstream_inference_cost` (actual provider cost, for BYOK requests only).

## Implementation Benefits

The documentation highlights that this approach offers "efficiency," "accuracy," and "transparency" by eliminating separate API calls while using native tokenizers for precision.

## Code Examples

The guide provides implementations in TypeScript and Python using both OpenRouter SDK and OpenAI SDK, demonstrating both standard and streaming request patterns.

## Alternative Method

Users can retrieve usage asynchronously by saving the `id` field from responses and querying the `/generation` endpoint later.
