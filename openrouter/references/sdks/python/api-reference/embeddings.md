# Embeddings - Python SDK Documentation

## Overview

The OpenRouter Python SDK provides text embedding endpoints for generating embeddings through an embeddings router. The SDK is currently in beta, with issues reported on GitHub.

## Available Operations

Two main operations are supported:

1. **generate** - Submit an embedding request
2. **list_models** - List all available embeddings models

## Generate Operation

Submits an embedding request to the embeddings router.

### Example Code

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.embeddings.generate(input="<value>", model="Taurus")
    print(res)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input` | InputUnion | Yes | Input text for embedding |
| `model` | string | Yes | Model selection |
| `encoding_format` | EncodingFormat (Optional) | No | Format specification |
| `dimensions` | integer (Optional) | No | Dimension setting |
| `user` | string (Optional) | No | User identifier |
| `provider` | ProviderPreferences (Optional) | No | Provider routing preferences |
| `input_type` | string (Optional) | No | Input type specification |
| `retries` | RetryConfig (Optional) | No | Retry configuration override |

### Response & Errors

Returns `CreateEmbeddingsResponse`. Possible errors include 400 (Bad Request), 401 (Unauthorized), 402 (Payment Required), 404 (Not Found), 429 (Too Many Requests), 500 (Internal Server), 502 (Bad Gateway), 503 (Service Unavailable), 524 (Edge Timeout), and 529 (Provider Overloaded).

## List Models Operation

Returns available embeddings models and properties.

### Example Code

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.embeddings.list_models()
    print(res)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `retries` | RetryConfig (Optional) | No | Retry configuration override |

### Response & Errors

Returns `ModelsListResponse`. Possible errors include 400 (Bad Request), 500 (Internal Server), and general 4XX/5XX errors.
