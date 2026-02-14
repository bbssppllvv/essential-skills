# Prompt Caching Documentation

## Overview

OpenRouter enables prompt caching across multiple AI providers to reduce inference costs. Most providers automatically enable caching, though some require per-message configuration.

## Cache Usage Inspection

Check cache savings through:
- Activity page detail buttons
- `/api/v1/generation` API endpoint
- `prompt_tokens_details` object in API responses

The `cache_discount` field shows cost reduction. Providers like Anthropic charge for cache writes but discount reads.

### Usage Metrics

```json
{
  "usage": {
    "prompt_tokens": 10339,
    "completion_tokens": 60,
    "prompt_tokens_details": {
      "cached_tokens": 10318,
      "cache_write_tokens": 0
    }
  }
}
```

Key fields:
- `cached_tokens`: Tokens read from cache
- `cache_write_tokens`: Tokens written to cache on first request

## Provider-Specific Details

### OpenAI
- Cache writes: Free
- Cache reads: 0.25x–0.50x base input price
- Automated, no configuration needed
- Minimum 1024 tokens required

### Anthropic Claude
- Cache writes (5-min): ~1.25x base price
- Cache writes (1-hour): 2x base price
- Cache reads: Discounted pricing
- Requires `cache_control` breakpoints (max 4)
- Recommended for large content blocks

**Supported models:** Claude Opus 4.5/4.1/4, Sonnet 4.5/4, Haiku 4.5/3.5

**TTL options:**
```json
"cache_control": { "type": "ephemeral" }  // 5 minutes
"cache_control": { "type": "ephemeral", "ttl": "1h" }  // 1 hour
```

### Google Gemini
- Implicit caching available (Gemini 2.5 Pro/Flash)
- No write/storage costs
- Cache reads: Discounted pricing
- 3–5 minute TTL
- Minimum token thresholds per model
- Requires explicit `cache_control` breakpoints

### DeepSeek, Grok, Moonshot, Groq
- Automated caching
- No write costs (except DeepSeek)
- Discounted cache reads

## Implementation Example

```json
{
  "messages": [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": "You are a historian studying Roman history."
        },
        {
          "type": "text",
          "text": "LARGE TEXT CONTENT",
          "cache_control": { "type": "ephemeral" }
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "What triggered collapse?"
        }
      ]
    }
  ]
}
```
