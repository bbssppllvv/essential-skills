# Update an API Key

## Endpoint Overview

The PATCH endpoint at `https://openrouter.ai/api/v1/keys/{hash}` allows modification of existing API keys. A management key is required for authentication.

## Request Details

**Method:** PATCH
**Content-Type:** application/json

### Path Parameters
- `hash` (string, required): Unique identifier for the API key to modify

### Headers
- `Authorization` (required): Bearer token using a management API key

### Request Body

Optional fields that can be updated:

| Field | Type | Description |
|-------|------|-------------|
| name | string | New display name for the key |
| disabled | boolean | Enable or disable the key |
| limit | number or null | USD spending limit |
| limit_reset | string or null | Reset frequency: "daily", "weekly", "monthly" |
| include_byok_in_limit | boolean | Include BYOK usage in spending limits |

## Response

**Status 200:** Returns the updated API key object containing:
- hash, name, label, disabled status
- limit and remaining balance information
- Usage metrics (total, daily, weekly, monthly)
- BYOK usage breakdown
- Timestamps (created_at, updated_at, expires_at)

**Error Responses:**
- 400: Invalid request parameters
- 401: Missing or invalid authentication
- 404: API key not found
- 429: Rate limit exceeded
- 500: Server error

## Code Examples

Available implementations provided in: Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. Each demonstrates setting a new name, disabling state, spending limit, reset frequency, and BYOK inclusion setting.
