# Generations - Python SDK

## Overview

This documentation covers the generation history endpoints available in the OpenRouter Python SDK, which is currently in beta. Users can report issues on GitHub.

## Available Operations

- **get_generation** - Retrieves request and usage metadata for a generation

## get_generation Method

### Purpose

Fetches metadata about a specific generation, including request details and usage information.

### Code Example

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.generations.get_generation(id="<id>")

    # Handle response
    print(res)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | The generation identifier |
| `retries` | Optional[utils.RetryConfig] | No | Retry configuration override |

### Response

Returns a `GetGenerationResponse` object containing the generation metadata.

### Possible Error Responses

| Error Type | Status Code | Format |
|------------|-------------|--------|
| UnauthorizedResponseError | 401 | application/json |
| PaymentRequiredResponseError | 402 | application/json |
| NotFoundResponseError | 404 | application/json |
| TooManyRequestsResponseError | 429 | application/json |
| InternalServerResponseError | 500 | application/json |
| BadGatewayResponseError | 502 | application/json |
| EdgeNetworkTimeoutResponseError | 524 | application/json |
| ProviderOverloadedResponseError | 529 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |
