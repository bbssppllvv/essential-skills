# Bulk Unassign Members from a Guardrail

## Endpoint Overview

This API endpoint removes multiple organization members from a specific guardrail assignment.

**Endpoint:** `POST https://openrouter.ai/api/v1/guardrails/{id}/assignments/members/remove`

**Authentication:** Management API key required via Bearer token

## Request Parameters

| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|
| id | Path | UUID string | Yes | Unique identifier for the guardrail |
| Authorization | Header | String | Yes | Bearer token with management key |
| member_user_ids | Body | Array of strings | Yes | User IDs to unassign from guardrail |

## Request Body

```json
{
  "member_user_ids": ["user_abc123", "user_def456"]
}
```

## Response

**Success (200):**
```json
{
  "unassigned_count": 2
}
```

The response includes the count of successfully unassigned members.

## Error Responses

- **400:** Invalid input provided
- **401:** Missing or invalid authentication credentials
- **404:** Specified guardrail not found
- **500:** Server error

## Code Examples

Examples are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. Each demonstrates setting the Authorization header with a bearer token and posting the member_user_ids array to the endpoint.
