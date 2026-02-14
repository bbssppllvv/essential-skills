# List API Keys - OpenRouter Documentation

## Endpoint Overview

The List API Keys endpoint retrieves all API keys associated with an authenticated user.

**Endpoint:** `GET https://openrouter.ai/api/v1/keys`

**Authentication:** Requires a management API key as a bearer token in the Authorization header.

## Query Parameters

- **include_disabled** (optional): Boolean to include disabled API keys in results
- **offset** (optional): Number of keys to skip for pagination

## Response Schema

The endpoint returns a JSON object containing a `data` array. Each API key object includes:

**Key Properties:**
- `hash`: Unique identifier for the key
- `name`: Key name
- `label`: Human-readable label
- `disabled`: Boolean status
- `limit` / `limit_remaining`: Spending limits in USD
- `usage`, `usage_daily`, `usage_weekly`, `usage_monthly`: Credit usage metrics
- `byok_usage*`: External BYOK usage metrics
- `created_at`, `updated_at`, `expires_at`: Timestamp fields

## HTTP Status Codes

- **200**: Successful response with key list
- **401**: Authentication failure
- **429**: Rate limit exceeded
- **500**: Server error

## Code Examples

The documentation provides implementation examples in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. All examples follow the pattern of sending a GET request with Bearer token authentication to the endpoint URL.
