# Get Remaining Credits API Documentation

## Endpoint Overview

The GET `/api/v1/credits` endpoint retrieves credit information for authenticated users. This requires a management API key for authorization.

## Request Details

**URL:** `https://openrouter.ai/api/v1/credits`

**Method:** GET

**Authentication:** Bearer token in Authorization header (management key required)

## Response Schema

The successful response returns an object containing:

- `total_credits` (number): Credits purchased by the user
- `total_usage` (number): Credits consumed to date

## Status Codes

| Code | Description |
|------|-------------|
| 200 | Successfully returns credit data |
| 401 | Missing or invalid authentication credentials |
| 403 | Non-management keys cannot access this endpoint |
| 500 | Server error occurred |

## Implementation Examples

The documentation provides code samples in multiple languages:

- **Python** using requests library
- **JavaScript** using Fetch API
- **Go** using net/http
- **Ruby** using Net::HTTP
- **Java** using Unirest
- **PHP** using Guzzle HTTP client
- **C#** using RestSharp
- **Swift** using URLSession

Each example follows the same pattern: sending a GET request with the bearer token in the Authorization header and parsing the JSON response.
