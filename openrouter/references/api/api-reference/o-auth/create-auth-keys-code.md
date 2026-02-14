# Create Authorization Code

## Overview

This endpoint generates an authorization code for OAuth2 PKCE flow, enabling creation of user-controlled API keys.

**Endpoint:** `POST https://openrouter.ai/api/v1/auth/keys/code`

## Request Details

### Headers
- `Authorization` (required): API key as bearer token
- `Content-Type`: application/json

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `callback_url` | string (URI) | Yes | "The callback URL to redirect to after authorization. Note, only https URLs on ports 443 and 3000 are allowed." |
| `code_challenge` | string | No | PKCE code challenge for enhanced security |
| `code_challenge_method` | string | No | Method for code challenge generation (S256 or plain) |
| `limit` | number | No | Credit limit for the generated API key |
| `expires_at` | string (date-time) | No | Optional expiration timestamp for the API key |

## Response

### Success (200)

Returns authorization code data:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | "The authorization code ID to use in the exchange request" |
| `app_id` | number | Application ID associated with auth code |
| `created_at` | string | ISO 8601 timestamp of creation |

### Error Responses
- **400**: Bad Request - Invalid parameters or malformed input
- **401**: Unauthorized - Missing or invalid credentials
- **500**: Internal Server Error

## Code Examples

Available implementations provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. All examples follow the same pattern: POST request with JSON payload and bearer token authentication.
