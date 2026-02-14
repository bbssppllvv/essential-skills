# Models - Python SDK Documentation

## Overview

The Models section provides endpoints for accessing model information within the OpenRouter Python SDK, currently in beta.

### Available Operations

- **count** - Retrieve the total number of available models
- **list** - Display all models with their associated properties
- **list_for_user** - Retrieve models based on user provider preferences, privacy settings, and guardrails

---

## count

Obtains the total count of available models.

### Code Example

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.models.count()

    # Handle response
    print(res)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `retries` | Optional[utils.RetryConfig] | No | Configuration to override default retry behavior |

### Response

**components.ModelsCountResponse**

### Error Handling

| Error Type | Status Code | Content Type |
|------------|-------------|--------------|
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |

---

## list

Displays all models and their properties.

### Code Example

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.models.list()

    # Handle response
    print(res)
```

### Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `category` | Optional[operations.Category] | No | Filter models by use case category | programming |
| `supported_parameters` | Optional[str] | No | N/A | |
| `retries` | Optional[utils.RetryConfig] | No | Configuration to override default retry behavior | |

### Response

**components.ModelsListResponse**

### Error Handling

| Error Type | Status Code | Content Type |
|------------|-------------|--------------|
| BadRequestResponseError | 400 | application/json |
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |

---

## list_for_user

Retrieves models filtered by user provider preferences, privacy settings, and guardrails. Results are filtered for EU in-region routing when using `eu.openrouter.ai/api/v1/...`.

### Code Example

```python
from openrouter import OpenRouter, operations
import os

with OpenRouter() as open_router:

    res = open_router.models.list_for_user(security=operations.ListModelsUserSecurity(
        bearer=os.getenv("OPENROUTER_BEARER", ""),
    ))

    # Handle response
    print(res)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `security` | operations.ListModelsUserSecurity | Yes | Security requirements for the request |
| `retries` | Optional[utils.RetryConfig] | No | Configuration to override default retry behavior |

### Response

**components.ModelsListResponse**

### Error Handling

| Error Type | Status Code | Content Type |
|------------|-------------|--------------|
| UnauthorizedResponseError | 401 | application/json |
| NotFoundResponseError | 404 | application/json |
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |

---

**Note:** "The Python SDK and docs are currently in beta. Report issues on GitHub."
