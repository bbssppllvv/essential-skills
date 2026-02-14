# Error Handling in Responses API Beta

## Overview

The Responses API Beta implements a standardized error response structure. This API is in beta stage and may have breaking changes.

Additionally, the API operates in a stateless manner, meaning each request is independent and no conversation state is persisted between requests.

## Error Response Format

When errors occur, the API returns a structured JSON response:

```json
{
  "error": {
    "code": "invalid_prompt",
    "message": "Detailed error description"
  },
  "metadata": null
}
```

## Error Codes Reference

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `invalid_prompt` | Request validation failed | 400 |
| `rate_limit_exceeded` | Too many requests | 429 |
| `server_error` | Internal server error | 500+ |

## Key Takeaway

Developers should anticipate three primary error categories when integrating with this API: validation failures returning 400-level responses, rate limiting at 429, and server-side issues at 500+. Users must include complete conversation history with each request since no state persists between interactions.
