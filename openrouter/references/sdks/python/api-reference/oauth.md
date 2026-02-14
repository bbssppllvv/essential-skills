# OAuth - Python SDK Documentation

## Overview

The OAuth authentication endpoints enable developers to exchange authorization codes for API keys using the PKCE flow, allowing users to maintain controlled access to their API credentials.

## Available Operations

1. **exchange_auth_code_for_api_key** - Converts an authorization code into a user-controlled API key
2. **create_auth_code** - Generates an authorization code for PKCE flow initiation

## exchange_auth_code_for_api_key

Exchanges an authorization code from the PKCE flow for a user-controlled API key.

### Example

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:
    res = open_router.o_auth.exchange_auth_code_for_api_key(
        code="auth_code_abc123def456"
    )
    print(res)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | str | Yes | Authorization code from OAuth redirect |
| `code_verifier` | Optional[str] | No | PKCE code verifier if applicable |
| `code_challenge_method` | Optional | No | Method for code challenge (e.g., S256) |
| `retries` | Optional[utils.RetryConfig] | No | Retry behavior configuration |

### Errors

- 400: BadRequestResponseError
- 403: ForbiddenResponseError
- 500: InternalServerResponseError

## create_auth_code

Generates an authorization code for the PKCE flow to establish a user-controlled API key.

### Example

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:
    res = open_router.o_auth.create_auth_code(
        callback_url="https://myapp.com/auth/callback"
    )
    print(res)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `callback_url` | str | Yes | HTTPS callback URL (ports 443, 3000 only) |
| `code_challenge` | Optional[str] | No | PKCE code challenge for security |
| `code_challenge_method` | Optional | No | Code challenge method (e.g., S256) |
| `limit` | Optional[float] | No | Credit limit for generated key |
| `expires_at` | Optional[date] | No | API key expiration date |
| `retries` | Optional[utils.RetryConfig] | No | Retry configuration |

### Errors

- 400: BadRequestResponseError
- 401: UnauthorizedResponseError
- 500: InternalServerResponseError

**Note:** The Python SDK is currently in beta. Report issues on GitHub.
