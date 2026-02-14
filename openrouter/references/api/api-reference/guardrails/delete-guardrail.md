# Delete a Guardrail API Documentation

## Endpoint Overview

The DELETE endpoint removes an existing guardrail from the OpenRouter API.

**Endpoint:** `DELETE https://openrouter.ai/api/v1/guardrails/{id}`

**Authentication:** Requires a management API key passed as a bearer token.

## Request Parameters

- **id** (path, required): A UUID identifying the guardrail to remove
- **Authorization** (header, required): Bearer token for authentication

## Response

**Success (200):** Returns a JSON object confirming deletion with a `deleted: true` property.

**Error Responses:**
- 401: Missing or invalid authentication credentials
- 404: The specified guardrail does not exist
- 500: Server-side error

## Code Examples

Implementations are provided for: Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. Each example demonstrates setting up the DELETE request with appropriate headers and handling the response.

All examples use the same URL structure with a sample UUID and require substituting the token placeholder with an actual management API key.
