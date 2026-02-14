# List Key Assignments for a Guardrail

## Endpoint Overview

The API endpoint enables retrieval of all API key assignments associated with a specific guardrail using a GET request to:

```
GET https://openrouter.ai/api/v1/guardrails/{id}/assignments/keys
```

A management API key is required for authentication.

## Request Parameters

**Path Parameter:**
- `id` (required): UUID format identifier for the guardrail

**Query Parameters:**
- `offset` (optional): Number of records to skip for pagination
- `limit` (optional): Maximum records to return (capped at 100)

**Header:**
- `Authorization` (required): Bearer token with management key

## Response Structure

**Success Response (200):**

The response contains an array of key assignment objects with these properties:

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Unique assignment identifier |
| key_hash | string | Hash of the assigned API key |
| guardrail_id | UUID | Associated guardrail ID |
| key_name | string | Name of the API key |
| key_label | string | Label of the API key |
| assigned_by | string/null | User ID who created the assignment |
| created_at | string | ISO 8601 timestamp |

The response also includes `total_count` indicating the total number of assignments.

**Error Responses:**
- 401: Authentication missing or invalid
- 404: Guardrail not found
- 500: Internal server error

## Code Examples

Complete implementation examples are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all demonstrating the same basic pattern: constructing a GET request with proper authorization headers.
