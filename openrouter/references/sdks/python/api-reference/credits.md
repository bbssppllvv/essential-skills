# Credits - Python SDK

## Overview

The credits section provides endpoints for managing credit accounts within the OpenRouter Python SDK.

### Available Operations

- **get_credits** - Retrieve remaining credits
- **create_coinbase_charge** - Generate a Coinbase charge for cryptocurrency payments

---

## get_credits

This method retrieves the total credits purchased and consumed for an authenticated user. A provisioning API key is required.

### Code Example

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.credits.get_credits()
    print(res)
```

### Parameters

| Parameter | Type | Required | Details |
|-----------|------|----------|---------|
| `retries` | Optional[utils.RetryConfig] | No | Override default retry behavior |

### Response Type

`operations.GetCreditsResponse`

### Potential Errors

| Error | Status Code | Format |
|-------|-------------|--------|
| UnauthorizedResponseError | 401 | application/json |
| ForbiddenResponseError | 403 | application/json |
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |

---

## create_coinbase_charge

Generates a Coinbase charge enabling cryptocurrency-based payments.

### Code Example

```python
from openrouter import OpenRouter, operations
import os

with OpenRouter() as open_router:

    res = open_router.credits.create_coinbase_charge(
        security=operations.CreateCoinbaseChargeSecurity(
            bearer=os.getenv("OPENROUTER_BEARER", ""),
        ),
        amount=100,
        sender="0x1234567890123456789012345678901234567890",
        chain_id=1
    )
    print(res)
```

### Parameters

| Parameter | Type | Required | Details |
|-----------|------|----------|---------|
| `security` | operations.CreateCoinbaseChargeSecurity | Yes | Authentication token |
| `amount` | float | Yes | Payment amount |
| `sender` | str | Yes | Wallet address |
| `chain_id` | components.ChainID | Yes | Blockchain network identifier |
| `retries` | Optional[utils.RetryConfig] | No | Override retry behavior |

### Response Type

`operations.CreateCoinbaseChargeResponse`

### Potential Errors

| Error | Status Code | Format |
|-------|-------------|--------|
| BadRequestResponseError | 400 | application/json |
| UnauthorizedResponseError | 401 | application/json |
| TooManyRequestsResponseError | 429 | application/json |
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |

---

**Note:** The Python SDK and documentation are currently in beta. Report issues on GitHub.
