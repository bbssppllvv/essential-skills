# Bulk Assign Keys to a Guardrail

## Endpoint Overview

**POST** `https://openrouter.ai/api/v1/guardrails/{id}/assignments/keys`

This endpoint enables bulk assignment of multiple API keys to a specific guardrail. A management key is required for authentication.

## Request Details

### Parameters

- **id** (path, required): Unique identifier of the guardrail (UUID format)
- **Authorization** (header, required): Management API key as bearer token

### Request Body

```json
{
  "key_hashes": ["string"]
}
```

The `key_hashes` field requires an array of API key hash strings to assign to the guardrail.

## Response

### Success (200)

```json
{
  "assigned_count": 0
}
```

Returns the number of keys successfully assigned.

### Error Responses

- **400**: Invalid input provided
- **401**: Missing or invalid authentication credentials
- **404**: Specified guardrail not found
- **500**: Server error

## Implementation Examples

Code samples are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. Each demonstrates the same operation: authenticating with a bearer token and posting an array of key hashes to assign them to the guardrail identified by UUID.
