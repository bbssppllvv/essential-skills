# Responses - Python SDK Documentation

## Overview

The `beta.responses` endpoint enables creation of streaming or non-streaming responses using the OpenResponses API format.

## Available Operations

- **send** - Create a response

## Send Method

Creates streaming or non-streaming responses via the OpenResponses API.

### Example Usage

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.beta.responses.send(service_tier="auto", stream=False)

    with res as event_stream:
        for event in event_stream:
            # handle event
            print(event, flush=True)
```

### Key Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `input` | OpenResponsesInput (optional) | Request input—string or array of items |
| `instructions` | string (optional) | Instruction text |
| `metadata` | Dict (optional) | Key-value pairs (max 16, keys ≤64 chars, values ≤512 chars) |
| `tools` | List (optional) | Tool definitions for requests |
| `model` / `models` | string / List (optional) | Model specification |
| `max_output_tokens` | float (optional) | Output token limit |
| `temperature` | float (optional) | Sampling temperature |
| `top_p` | float (optional) | Nucleus sampling parameter |
| `modalities` | List (optional) | "text" and/or "image" outputs |
| `stream` | bool (optional) | Enable streaming responses |
| `service_tier` | ServiceTier (optional) | Service tier selection |
| `user` | string (optional) | End-user identifier (max 128 chars) |
| `session_id` | string (optional) | Request grouping identifier (max 128 chars) |

### Response

Returns `operations.CreateResponsesResponse`

### Error Handling

The endpoint returns various HTTP status codes with `application/json` content:

- **400** - Bad Request
- **401** - Unauthorized
- **402** - Payment Required
- **404** - Not Found
- **429** - Too Many Requests
- **500** - Internal Server Error
- **503** - Service Unavailable

**Note:** The Python SDK remains in beta. Report issues on GitHub.
