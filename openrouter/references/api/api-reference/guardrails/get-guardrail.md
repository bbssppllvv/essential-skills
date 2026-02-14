# Get a Guardrail - API Documentation

## Overview

The Get a Guardrail endpoint retrieves a single guardrail configuration by its unique identifier from OpenRouter's API.

**Endpoint:** `GET https://openrouter.ai/api/v1/guardrails/{id}`

**Authentication:** Management API key required as a bearer token

## Request Parameters

- **id** (path, required): UUID format identifier for the guardrail
- **Authorization** (header, required): API key formatted as bearer token

## Response Details

### Success Response (200)

Returns guardrail details including:

- **id**: Unique UUID identifier
- **name**: Guardrail name
- **description**: Optional description text
- **limit_usd**: Optional spending limit in USD
- **reset_interval**: Optional reset frequency (daily, weekly, or monthly)
- **allowed_providers**: Optional list of provider IDs
- **allowed_models**: Optional array of model canonical_slugs
- **enforce_zdr**: Optional zero data retention enforcement flag
- **created_at**: ISO 8601 creation timestamp
- **updated_at**: Optional ISO 8601 last modification timestamp

### Error Responses

- **401**: Missing or invalid authentication credentials
- **404**: Specified guardrail does not exist
- **500**: Internal server error

## Code Examples

The documentation includes implementation examples in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all using the same request structure with appropriate HTTP client libraries for each language.
