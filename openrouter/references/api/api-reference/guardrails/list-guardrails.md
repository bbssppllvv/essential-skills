# List Guardrails API Documentation

## Endpoint Overview

The List Guardrails endpoint allows authenticated users to retrieve all guardrails associated with their account.

**Endpoint:** `GET https://openrouter.ai/api/v1/guardrails`

**Authentication:** Requires a management API key passed as a bearer token in the Authorization header.

## Query Parameters

- **offset** (optional, string): Specifies how many records to skip for pagination purposes
- **limit** (optional, string): Sets the maximum number of records returned, capped at 100

## Response Schema

The successful response (200) returns a JSON object containing:

- **data**: An array of guardrail objects with the following properties:
  - id (UUID): Unique guardrail identifier
  - name (string): Guardrail display name
  - description (string, nullable): Optional guardrail description
  - limit_usd (number, nullable): Spending limit in USD
  - reset_interval (string, nullable): Reset frequency (daily, weekly, or monthly)
  - allowed_providers (array, nullable): Permitted provider IDs
  - allowed_models (array, nullable): Approved model canonical slugs
  - enforce_zdr (boolean, nullable): Zero data retention enforcement flag
  - created_at (string): ISO 8601 creation timestamp
  - updated_at (string, nullable): ISO 8601 modification timestamp

- **total_count** (number): Total guardrails available

## Error Responses

- **401**: Authentication failure due to missing or invalid credentials
- **500**: Server-side error

## Code Examples

Available implementations provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all following the same pattern of sending a GET request with proper Authorization header.
