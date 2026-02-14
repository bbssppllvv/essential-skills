# APIKeys - Python SDK Documentation

## Overview
API key management endpoints for the OpenRouter Python SDK (currently in beta).

## Available Operations

- **list** - List API keys
- **create** - Create a new API key
- **update** - Update an API key
- **delete** - Delete an API key
- **get** - Get a single API key
- **get_current_key_metadata** - Get current API key

---

## list

Retrieve all API keys for the authenticated user. Requires a provisioning key.

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(api_key=os.getenv("OPENROUTER_API_KEY", "")) as open_router:
    res = open_router.api_keys.list(include_disabled="false", offset="0")
    print(res)
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `include_disabled` | Optional[str] | No | Whether to include disabled keys |
| `offset` | Optional[str] | No | Number of keys to skip for pagination |
| `retries` | Optional[utils.RetryConfig] | No | Override default retry behavior |

**Response:** `operations.ListResponse`

**Error Codes:** 401, 429, 500

---

## create

Generate a new API key for the authenticated user. Requires a provisioning key.

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(api_key=os.getenv("OPENROUTER_API_KEY", "")) as open_router:
    res = open_router.api_keys.create(name="My New API Key")
    print(res)
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | str | Yes | Name for the new API key |
| `limit` | OptionalNullable[float] | No | Spending limit in USD |
| `limit_reset` | OptionalNullable[operations.CreateKeysLimitReset] | No | Reset type (daily, weekly, monthly, or null) |
| `include_byok_in_limit` | Optional[bool] | No | Include BYOK usage in limit |
| `expires_at` | date | No | ISO 8601 UTC expiration timestamp |

**Response:** `operations.CreateKeysResponse`

**Error Codes:** 400, 401, 429, 500

---

## update

Modify an existing API key. Requires a provisioning key.

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(api_key=os.getenv("OPENROUTER_API_KEY", "")) as open_router:
    res = open_router.api_keys.update(hash="f01d526...")
    print(res)
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hash` | str | Yes | API key hash identifier |
| `name` | Optional[str] | No | New name |
| `disabled` | Optional[bool] | No | Disable the key |
| `limit` | OptionalNullable[float] | No | New spending limit |
| `limit_reset` | OptionalNullable[operations.UpdateKeysLimitReset] | No | New reset type |
| `include_byok_in_limit` | Optional[bool] | No | Include BYOK in limit |

**Response:** `operations.UpdateKeysResponse`

**Error Codes:** 400, 401, 404, 429, 500

---

## delete

Remove an API key. Requires a provisioning key.

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(api_key=os.getenv("OPENROUTER_API_KEY", "")) as open_router:
    res = open_router.api_keys.delete(hash="f01d526...")
    print(res)
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hash` | str | Yes | API key hash identifier |

**Response:** `operations.DeleteKeysResponse`

**Error Codes:** 401, 404, 429, 500

---

## get

Retrieve a single API key by hash. Requires a provisioning key.

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(api_key=os.getenv("OPENROUTER_API_KEY", "")) as open_router:
    res = open_router.api_keys.get(hash="f01d526...")
    print(res)
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hash` | str | Yes | API key hash identifier |

**Response:** `operations.GetKeyResponse`

**Error Codes:** 401, 404, 429, 500

---

## get_current_key_metadata

Fetch metadata about the API key used in the current authentication session.

**Example:**
```python
from openrouter import OpenRouter
import os

with OpenRouter(api_key=os.getenv("OPENROUTER_API_KEY", "")) as open_router:
    res = open_router.api_keys.get_current_key_metadata()
    print(res)
```

**Response:** `operations.GetCurrentKeyResponse`

**Error Codes:** 401, 500
