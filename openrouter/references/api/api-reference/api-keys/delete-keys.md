# Delete an API Key - OpenRouter Documentation

## Endpoint Overview

The Delete API Key endpoint allows removal of existing API keys from your OpenRouter account using a management key for authentication.

**HTTP Method:** DELETE
**URL:** `https://openrouter.ai/api/v1/keys/{hash}`

## Authentication Requirements

Management key required for this operation. The key must be passed as a bearer token in the Authorization header.

## Parameters

| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|
| hash | Path | String | Yes | The hash identifier of the API key to delete |
| Authorization | Header | String | Yes | API key as bearer token in Authorization header |

## Response Codes

- **200**: "API key deleted successfully" - Returns confirmation object
- **401**: Unauthorized - Missing or invalid authentication
- **404**: Not Found - API key does not exist
- **429**: Too Many Requests - Rate limit exceeded
- **500**: Internal Server Error

## Success Response Schema

```json
{
  "deleted": true
}
```

## Code Examples

Implementation examples are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all following this pattern:
1. Construct the DELETE request to the endpoint with the key's hash
2. Include the Authorization header with bearer token
3. Execute the request and handle the response

Each language example uses a sample hash for demonstration purposes.
