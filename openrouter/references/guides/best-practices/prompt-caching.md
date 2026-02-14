# Prompt Caching

## Overview

OpenRouter enables prompt caching across multiple AI providers to reduce inference costs. The feature works differently depending on the provider -- some handle it automatically, while others like Anthropic require explicit configuration per message.

## Cache Usage Inspection

Users can monitor caching effectiveness through three methods:

- The Activity page detail button
- The `/api/v1/generation` API endpoint
- The `prompt_tokens_details` object in API responses

The `cache_discount` field indicates cost savings, with providers like Anthropic showing negative discounts on writes but positive discounts on reads.

### Usage Metrics

```json
{
  "usage": {
    "prompt_tokens": 10339,
    "completion_tokens": 60,
    "total_tokens": 10399,
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
- Cache reads: 0.25x-0.50x base input price
- Automated, no configuration needed
- Minimum 1024 tokens required

### Anthropic Claude

- Cache writes (5-min TTL): ~1.25x base price
- Cache writes (1-hour TTL): 2x base price
- Cache reads: Discounted pricing
- Requires `cache_control` breakpoints (max 4 per request)
- Recommended for large content blocks

**Supported models:** Claude Opus 4.5/4.1/4, Sonnet 4.5/4, Haiku 4.5/3.5

**Minimum cacheable lengths:** Range from 1024 to 4096 tokens depending on the model.

#### System Message Caching (5-minute TTL)

```json
{
  "messages": [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": "You are a historian studying the fall of the Roman Empire. You know the following book very well:"
        },
        {
          "type": "text",
          "text": "HUGE TEXT BODY",
          "cache_control": {
            "type": "ephemeral"
          }
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "What triggered the collapse?"
        }
      ]
    }
  ]
}
```

#### User Message Caching (1-hour TTL)

```json
{
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Given the book below:"
        },
        {
          "type": "text",
          "text": "HUGE TEXT BODY",
          "cache_control": {
            "type": "ephemeral",
            "ttl": "1h"
          }
        },
        {
          "type": "text",
          "text": "Name all the characters in the above book"
        }
      ]
    }
  ]
}
```

### Google Gemini

- Implicit caching available (Gemini 2.5 Pro/Flash)
- No write/storage costs
- Cache reads: Discounted pricing
- 3-5 minute TTL
- Minimum token thresholds per model
- Supports explicit `cache_control` breakpoints

#### System Message Caching Example

```json
{
  "messages": [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": "You are a historian studying the fall of the Roman Empire. Below is an extensive reference book:"
        },
        {
          "type": "text",
          "text": "HUGE TEXT BODY HERE",
          "cache_control": {
            "type": "ephemeral"
          }
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "What triggered the collapse?"
        }
      ]
    }
  ]
}
```

#### User Message Caching Example

```json
{
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Based on the book text below:"
        },
        {
          "type": "text",
          "text": "HUGE TEXT BODY HERE",
          "cache_control": {
            "type": "ephemeral"
          }
        },
        {
          "type": "text",
          "text": "List all main characters mentioned in the text above."
        }
      ]
    }
  ]
}
```

### DeepSeek

- Automated caching
- Discounted cache reads
- Write costs apply

### Grok

- Automated caching
- No write costs
- Discounted cache reads

### Moonshot AI

- Automated caching
- No write costs
- Discounted cache reads

### Groq

- Automated caching
- No write costs
- Discounted cache reads

## Best Practices

- Keep consistent message array prefixes across requests to maximize cache hits
- Place dynamic elements toward the end of the messages array
- Reserve breakpoints for substantial content like character cards, CSV data, RAG materials, or book chapters
