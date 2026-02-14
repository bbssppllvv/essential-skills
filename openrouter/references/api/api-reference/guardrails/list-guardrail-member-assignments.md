# List Member Assignments for a Guardrail

## Overview

This endpoint retrieves all organization member assignments for a specific guardrail using the OpenRouter API.

**Endpoint:** `GET https://openrouter.ai/api/v1/guardrails/{id}/assignments/members`

**Authentication:** Requires a management API key as a bearer token

## Request Parameters

| Parameter | Type | Location | Required | Description |
|-----------|------|----------|----------|-------------|
| `id` | UUID | Path | Yes | Unique identifier of the guardrail |
| `offset` | String | Query | No | Number of records to skip for pagination |
| `limit` | String | Query | No | Maximum number of records to return (max 100) |
| `Authorization` | String | Header | Yes | API key as bearer token in Authorization header |

## Response

**Success (200):** Returns a JSON object containing:
- `data`: Array of member assignment objects
- `total_count`: Total number of member assignments

Each assignment object includes:
- `id`: Unique assignment identifier (UUID)
- `user_id`: Clerk user ID of the assigned member
- `organization_id`: Organization ID
- `guardrail_id`: Associated guardrail ID
- `assigned_by`: User ID who created the assignment
- `created_at`: ISO 8601 timestamp of creation

**Error Responses:**
- `401`: Missing or invalid authentication
- `404`: Guardrail not found
- `500`: Internal server error

## Code Examples

Language-specific implementation examples are provided for Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift.
