# Get Total Count of Available Models

## Endpoint Overview

The `/models/count` endpoint retrieves the total number of models available through OpenRouter's API using a GET request to `https://openrouter.ai/api/v1/models/count`.

## Authentication

This endpoint requires bearer token authentication via the `Authorization` header. The API key must be provided in the format: `Bearer <token>`.

## API Response

**Success Response (200):**
The endpoint returns a JSON object with a nested data structure:

```json
{
  "data": {
    "count": <number>
  }
}
```

The `count` property contains the total quantity of available models as a numeric value.

**Error Response (500):**
Internal server errors return a 500 status code without content.

## Implementation Examples

Code samples are provided in multiple programming languages:

- **Python**: Uses the `requests` library for HTTP GET operations
- **JavaScript**: Implements the endpoint using the Fetch API
- **Go**: Demonstrates usage with the standard `net/http` package
- **Ruby**: Shows implementation with the built-in `Net::HTTP` module
- **Java**: Uses the Unirest HTTP client library
- **PHP**: Implements the request through Guzzle HTTP client
- **C#**: Demonstrates usage with the RestSharp library
- **Swift**: Shows implementation using Foundation's URLSession

All examples follow the same pattern: constructing a GET request with proper authorization headers and processing the JSON response.
