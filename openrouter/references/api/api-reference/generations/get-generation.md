# Get Request & Usage Metadata for a Generation

## Endpoint Overview

The GET endpoint at `https://openrouter.ai/api/v1/generation` retrieves request and usage metadata for a specific generation.

## Parameters

**Query Parameters:**
- `id` (required, string): The unique generation identifier

**Headers:**
- `Authorization` (required, string): API key provided as a bearer token

## Response Details

A successful 200 response returns generation data including:

- **Identification**: Generation ID, upstream ID, model used
- **Costs**: Total cost, cache discount, upstream inference cost (all in USD)
- **Timing**: Creation timestamp, latency, moderation latency, generation time
- **Tokens**: Prompt tokens, completion tokens, cached tokens, reasoning tokens
- **Status**: Finish reason, whether streamed or cancelled
- **Provider Info**: Provider name, BYOK usage, provider responses (including fallbacks)
- **Additional Metadata**: App ID, external user identifier, router type, media/audio counts

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success—metadata returned |
| 401 | Authentication failed or missing |
| 402 | Insufficient credits/quota |
| 404 | Generation not found |
| 429 | Rate limit exceeded |
| 500 | Server error |
| 502 | Provider/upstream API failure |

## Code Examples

Examples provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all demonstrating the basic GET request pattern with proper authentication headers.
