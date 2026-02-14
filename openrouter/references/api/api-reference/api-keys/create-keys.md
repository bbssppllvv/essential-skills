# Create a new API key

## Endpoint Overview

POST `https://openrouter.ai/api/v1/keys`

The endpoint allows authenticated users to generate a new API key. A management key is required for this operation.

## Request Parameters

The request body accepts the following JSON properties:

- **name** (required, string): Identifier for the new API key
- **limit** (optional, number): Maximum spending threshold in USD
- **limit_reset** (optional): Frequency for limit resets--choose from "daily", "weekly", "monthly", or null. Resets occur at midnight UTC
- **include_byok_in_limit** (optional, boolean): Controls whether external BYOK usage counts toward the spending limit
- **expires_at** (optional, ISO 8601 string): UTC expiration timestamp for the key; non-UTC timezones are rejected

Authentication uses a bearer token in the Authorization header.

## Response Details

A successful 201 response includes:

- **key**: The actual API key string (displayed only once)
- **data**: Complete key metadata including:
  - hash, name, label, disabled status
  - Current and remaining limits
  - Usage metrics (total, daily, weekly, monthly for both standard and BYOK)
  - Creation and update timestamps
  - Expiration information

## HTTP Status Codes

- 201: Key created successfully
- 400: Invalid request parameters
- 401: Missing or invalid authentication
- 429: Rate limit exceeded
- 500: Server error

## Code Examples

Examples are provided in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all demonstrating creation of a key named "Analytics Service Key" with a $150 monthly limit expiring June 30, 2028.
