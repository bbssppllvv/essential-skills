# Get User Activity Grouped by Endpoint

## Overview

The endpoint at `GET https://openrouter.ai/api/v1/activity` retrieves "user activity data grouped by endpoint for the last 30 (completed) UTC days." A management API key is required for authentication.

## Parameters

**Query Parameters:**
- `date` (optional): Filter results to a specific UTC date within the last 30 days using YYYY-MM-DD format

**Headers:**
- `Authorization` (required): Bearer token authentication using a management key

## Response Schema

The API returns activity items with the following properties:

| Field | Type | Description |
|-------|------|-------------|
| date | string | Activity date in YYYY-MM-DD format |
| model | string | Model identifier (e.g., "openai/gpt-4.1") |
| model_permaslug | string | Permanent model version identifier |
| endpoint_id | string | Unique endpoint identifier |
| provider_name | string | Service provider name |
| usage | number | Total cost in USD (OpenRouter credits) |
| byok_usage_inference | number | External credits spent in USD |
| requests | number | Request count |
| prompt_tokens | number | Total prompt tokens |
| completion_tokens | number | Total completion tokens |
| reasoning_tokens | number | Total reasoning tokens |

## HTTP Status Codes

- **200**: Successful response with activity data
- **400**: Invalid date format or out-of-range date
- **401**: Missing or invalid credentials
- **403**: Non-management key provided
- **500**: Server error

## Code Examples

SDK implementations are provided for: Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift.
