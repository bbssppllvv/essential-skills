# OpenRouter API Reference Documentation

## Overview

OpenRouter normalizes API schemas across multiple AI model providers, allowing developers to use a single interface. The request and response structures closely mirror OpenAI's Chat API with minor modifications.

## OpenAPI Specification

Complete API documentation is available in standardized formats:
- **YAML**: https://openrouter.ai/openapi.yaml
- **JSON**: https://openrouter.ai/openapi.json

These can be used with tools like Swagger UI, Postman, or OpenAPI code generators.

## Request Structure

Requests are sent as `POST` to `/api/v1/chat/completions` with a JSON body containing:

**Core Parameters:**
- `messages` or `prompt` (at least one required)
- `model` (optional; uses user default if unspecified)
- `response_format` (for structured JSON outputs)
- `stream` (boolean; enables streaming)

**Model Control:**
- `max_tokens`: Range [1, context_length)
- `temperature`: Range [0, 2]
- `stop`: String or array of strings
- `seed`: Integer only

**Advanced Options:**
- `top_p`, `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`
- `logit_bias`, `top_logprobs`, `min_p`, `top_a`
- `prediction` (for latency optimization)

**Tool Integration:**
- `tools`: Array of tool definitions
- `tool_choice`: 'none', 'auto', or specific function selection

**OpenRouter-Specific:**
- `plugins`: Web search, PDF parsing, response healing
- `transforms`: Prompt transformation options
- `models`: Multiple model fallback routing
- `route`: 'fallback' strategy
- `provider`: Provider preferences
- `user`: Stable identifier for abuse detection
- `debug`: Echo upstream request body

## Response Format

Responses follow this structure:

```
{
  "id": "generation-identifier",
  "choices": [{
    "finish_reason": "normalized-reason",
    "native_finish_reason": "provider-specific-reason",
    "message": { "role": "assistant", "content": "..." }
  }],
  "usage": {
    "prompt_tokens": number,
    "completion_tokens": number,
    "total_tokens": number
  },
  "model": "provider/model-name"
}
```

**Finish Reasons:** Normalized to `tool_calls`, `stop`, `length`, `content_filter`, or `error`.

**Token Usage:** Calculated using the model's native tokenizer. Streaming returns usage in the final chunk.

## Key Features

**Structured Outputs:** Use `response_format` for JSON mode or strict schema validation.

**Plugins:** Enable capabilities like real-time web search (`web`), document processing (`file-parser`), and automatic error correction (`response-healing`).

**Assistant Prefill:** Include an assistant message at the end of the conversation to guide model responses.

**Headers:** Optional metadata headers identify your application:
- `HTTP-Referer`: Your site URL
- `X-Title`: Your application name

**Model Routing:** Omitting the model parameter uses the user's default. Otherwise, OpenRouter selects optimal infrastructure and implements fallback strategies.

**Streaming:** Server-Sent Events (SSE) supported by setting `stream: true`. Ignore comment payloads in the stream.

## Generation Stats

Query historical generation data using the `/api/v1/generation` endpoint with the returned generation ID for auditing and asynchronous stat retrieval.
