# Bulk Assign Members to a Guardrail

## Endpoint Overview

**Method:** POST
**URL:** `https://openrouter.ai/api/v1/guardrails/{id}/assignments/members`
**Content-Type:** application/json

This endpoint allows you to assign multiple organization members to a specific guardrail. A management API key is required for authentication.

## Parameters

| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|
| id | path | string (uuid) | Yes | The guardrail's unique identifier |
| Authorization | header | string | Yes | Bearer token with management key |

## Request Body

```json
{
  "member_user_ids": ["user_abc123", "user_def456"]
}
```

**Required field:** `member_user_ids` (array of strings) -- The list of member user IDs to assign

## Response

**Success (200):**
```json
{
  "assigned_count": 2
}
```

Returns the number of members successfully assigned to the guardrail.

**Error Responses:**
- 400: Invalid input
- 401: Missing or invalid authentication
- 404: Guardrail not found
- 500: Server error

## SDK Examples

Code samples are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. Each example demonstrates posting to the endpoint with the required authorization header and member user IDs in the request body.
