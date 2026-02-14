# Bulk Unassign Keys from a Guardrail

## Endpoint Overview

This API endpoint removes multiple API keys from a specific guardrail through a POST request to `https://openrouter.ai/api/v1/guardrails/{id}/assignments/keys/remove`. A management key is required for authentication.

## Request Parameters

**Path Parameter:**
- `id` (string, UUID format): The unique identifier for the target guardrail

**Header:**
- `Authorization` (required): Bearer token using a management API key

**Request Body:**
- `key_hashes` (array, required): List of API key hashes to remove from the guardrail

## Response

**Success (200):**
Returns an object containing `unassigned_count` (number), indicating how many keys were successfully removed.

**Error Codes:**
- `400`: Invalid input provided
- `401`: Missing or invalid authentication credentials
- `404`: Specified guardrail does not exist
- `500`: Server-side error

## Code Examples

SDK implementations are provided in multiple languages including Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. Each example demonstrates posting to the endpoint with a sample guardrail UUID and key hash, passing the required authorization header and JSON content type.
