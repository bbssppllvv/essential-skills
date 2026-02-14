# Analytics - Python SDK Documentation

## Overview
The Analytics endpoint provides user activity data grouped by endpoint for the last 30 completed UTC days. A provisioning key is required to access this functionality.

## Available Operations
- `get_user_activity` - Retrieves user activity grouped by endpoint

## get_user_activity Method

### Purpose
Returns user activity data grouped by endpoint for the last 30 completed UTC days, requiring a provisioning key.

### Code Example
```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.analytics.get_user_activity(date_="2025-08-24")

    # Handle response
    print(res)
```

### Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `date_` | Optional[str] | No | Filter by a single UTC date in the last 30 days (YYYY-MM-DD format) | 2025-08-24 |
| `retries` | Optional[utils.RetryConfig] | No | Configuration to override default retry behavior | -- |

### Response
Returns `operations.GetUserActivityResponse`

### Possible Error Responses

| Error Type | Status Code | Content Type |
|-----------|-------------|--------------|
| BadRequestResponseError | 400 | application/json |
| UnauthorizedResponseError | 401 | application/json |
| ForbiddenResponseError | 403 | application/json |
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |

---

**Note:** The Python SDK and documentation are currently in beta. Issues can be reported on GitHub.
