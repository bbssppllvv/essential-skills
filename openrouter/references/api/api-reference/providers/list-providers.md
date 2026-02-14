# List All Providers - OpenRouter API Documentation

## Endpoint Overview

The "List all providers" endpoint enables users to retrieve a comprehensive list of available AI model providers through OpenRouter's API infrastructure.

**Request Method:** GET
**URL:** `https://openrouter.ai/api/v1/providers`

## Authentication

The endpoint requires API key authentication via Bearer token in the Authorization header.

## Response Structure

Successful requests return a 200 status code with provider data containing:

- **name:** Display identifier for the provider
- **slug:** URL-friendly provider identifier
- **privacy_policy_url:** Link to privacy documentation (nullable)
- **terms_of_service_url:** Link to service terms (nullable)
- **status_page_url:** Link to operational status page (nullable)

The response wraps these items in a data array object.

## Error Handling

The API returns a 500 status code for unexpected server errors.

## Implementation Examples

Code samples are provided for multiple programming languages:

- **Python:** Using the requests library
- **JavaScript:** Using the Fetch API
- **Go:** Using net/http package
- **Ruby:** Using Net::HTTP
- **Java:** Using Unirest HTTP client
- **PHP:** Using Guzzle HTTP client
- **C#:** Using RestSharp
- **Swift:** Using Foundation URLSession

All examples follow the pattern of adding an Authorization header with a Bearer token and handling the JSON response appropriately.
