# Get a Single API Key

## Endpoint Overview

This documentation describes the OpenRouter API endpoint for retrieving details about a specific API key using its hash identifier.

**Endpoint:** `GET https://openrouter.ai/api/v1/keys/{hash}`

**Authentication Required:** A management key is needed to access this endpoint.

## Request Parameters

- **Path Parameter - `hash`:** The unique identifier for the API key to retrieve (string, required)
- **Header - `Authorization`:** Bearer token for authentication (required)

## Response Details

### Success Response (HTTP 200)

The endpoint returns a JSON object containing API key information, including:

- **Identification:** hash, name, and label
- **Status:** disabled flag and expiration timestamp
- **Spending Controls:** credit limits and remaining balance
- **Usage Metrics:** Total, daily, weekly, and monthly usage in USD
- **BYOK Metrics:** External "Bring Your Own Key" usage tracking across time periods
- **Timestamps:** Creation and last update dates in ISO 8601 format

### Error Responses

- **401:** Authentication credentials are missing or invalid
- **404:** The specified API key does not exist
- **429:** Rate limit has been exceeded
- **500:** Server error occurred

## Code Examples

The documentation provides implementation examples in 8 programming languages:

Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift

Each example demonstrates the same pattern: constructing a GET request to the endpoint with the bearer token in the Authorization header and displaying the JSON response.
