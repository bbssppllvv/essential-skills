# Providers - Python SDK

## Overview

This documentation covers provider information endpoints available in the OpenRouter Python SDK, which is currently in beta. Users encountering issues are encouraged to report them on GitHub.

## Available Operations

- **list** - Retrieve all providers

## List Operation

### Description

Fetches a comprehensive list of all available providers.

### Code Example

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.providers.list()
    print(res)
```

### Parameters

| Parameter | Type | Required | Purpose |
|-----------|------|----------|---------|
| `retries` | Optional[utils.RetryConfig] | No | Override default client retry settings |

### Response

Returns: `operations.ListProvidersResponse`

### Possible Errors

| Error Type | Status Code | Format |
|------------|-------------|--------|
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |
