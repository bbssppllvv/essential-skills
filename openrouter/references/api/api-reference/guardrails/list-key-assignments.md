# List All Key Assignments API Documentation

## Endpoint Overview

The endpoint `GET https://openrouter.ai/api/v1/guardrails/assignments/keys` retrieves all API key guardrail assignments for an authenticated user. A management key is required for authorization.

## Request Parameters

**Query Parameters:**
- `offset` (optional): Pagination parameter specifying records to skip
- `limit` (optional): Maximum records returned per request, capped at 100

**Header:**
- `Authorization` (required): Bearer token using a management API key

## Response Format

A successful 200 response contains:
- `data`: An array of assignment objects
- `total_count`: Total number of assignments available

Each assignment object includes:
- `id`: UUID identifier
- `key_hash`: Hashed API key value
- `guardrail_id`: UUID of associated guardrail
- `key_name` and `key_label`: Key identification details
- `assigned_by`: User ID of who created the assignment
- `created_at`: ISO 8601 timestamp

## Error Responses

- `401`: Authentication missing or invalid
- `500`: Server error

## Code Examples

Implementation examples are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all following the same pattern of sending a GET request with proper authorization headers.
