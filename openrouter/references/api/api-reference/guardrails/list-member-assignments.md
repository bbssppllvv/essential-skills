# List All Member Assignments

## Endpoint Overview

The GET endpoint at `https://openrouter.ai/api/v1/guardrails/assignments/members` retrieves all organization member guardrail assignments for the authenticated user. A management API key is required for access.

## Parameters

**Query Parameters:**
- `offset` (optional): Records to skip for pagination
- `limit` (optional): Maximum records to return (capped at 100)

**Headers:**
- `Authorization` (required): API key provided as a bearer token

## Response Structure

Successful requests return a 200 status with:
- `data`: Array of member assignment objects containing:
  - `id`: Unique assignment identifier (UUID)
  - `user_id`: Clerk user ID of the assigned member
  - `organization_id`: Organization identifier
  - `guardrail_id`: Guardrail identifier (UUID)
  - `assigned_by`: User ID of the person who created the assignment
  - `created_at`: ISO 8601 timestamp of assignment creation

- `total_count`: Total number of member assignments available

## Error Responses

- **401**: Unauthorized -- Missing or invalid authentication
- **500**: Internal Server Error

## Code Examples

Complete implementations are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all following the same pattern: constructing a GET request with proper authorization headers to the endpoint.
