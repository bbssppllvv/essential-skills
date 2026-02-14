# Create a Guardrail API Documentation

## Endpoint Overview

**POST** `https://openrouter.ai/api/v1/guardrails`

Creates a new guardrail for an authenticated user. A management API key is required.

## Authentication

The endpoint requires an API key passed as a bearer token in the Authorization header.

## Request Body

The following parameters can be included in the JSON request:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Name for the new guardrail |
| `description` | string or null | No | Description of the guardrail |
| `limit_usd` | number or null | No | Spending limit in USD |
| `reset_interval` | string or null | No | Limit reset frequency (daily, weekly, or monthly) |
| `allowed_providers` | array or null | No | List of allowed provider IDs |
| `allowed_models` | array or null | No | Model identifiers (slug or canonical_slug accepted) |
| `enforce_zdr` | boolean or null | No | Enable zero data retention enforcement |

## Response

**Status Code 201:** Successful creation

Returns a JSON object containing the created guardrail with these fields:

- `id`: UUID identifier
- `name`: Guardrail name
- `description`: Optional description
- `limit_usd`: USD spending cap
- `reset_interval`: Reset frequency
- `allowed_providers`: Provider list
- `allowed_models`: Array of model canonical_slugs
- `enforce_zdr`: Data retention setting
- `created_at`: ISO 8601 creation timestamp
- `updated_at`: ISO 8601 last update timestamp (nullable)

## Code Examples

Complete implementation examples provided for Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift programming languages.
