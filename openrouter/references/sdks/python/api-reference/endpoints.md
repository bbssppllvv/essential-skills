# Endpoints - Python SDK Documentation

## Overview

This documentation covers endpoint information methods within the OpenRouter Python SDK, which is currently in beta.

## Available Operations

The Endpoints API provides two main operations:

1. **list** - Retrieves all available endpoints for a specified model
2. **list_zdr_endpoints** - Shows how Zero Downtime Routing (ZDR) affects available endpoints

## list Method

**Purpose:** Retrieve endpoints for a model

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:
    res = open_router.endpoints.list(author="<value>", slug="<value>")
    print(res)
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| author | str | Yes | Model author identifier |
| slug | str | Yes | Model slug identifier |
| retries | Optional[utils.RetryConfig] | No | Custom retry configuration |

**Response:** ListEndpointsResponse object

**Possible Errors:** 404 (Not Found), 500 (Internal Server Error), or general 4XX/5XX errors

## list_zdr_endpoints Method

**Purpose:** Preview Zero Downtime Routing impact on endpoint availability

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:
    res = open_router.endpoints.list_zdr_endpoints()
    print(res)
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| retries | Optional[utils.RetryConfig] | No | Custom retry configuration |

**Response:** ListEndpointsZdrResponse object

**Possible Errors:** 500 (Internal Server Error) or general 4XX/5XX errors
