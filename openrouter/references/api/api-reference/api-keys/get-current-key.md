# Get Current API Key Endpoint Documentation

## Overview

The endpoint retrieves details about the API key associated with the current authentication session.

**Endpoint:** `GET https://openrouter.ai/api/v1/key`

**Purpose:** Get information on the API key associated with the current authentication session.

## Authentication

This endpoint requires bearer token authentication via the Authorization header.

## Response Details

A successful request returns a 200 status with API key information including:

- **Label:** Human-readable identifier for the key
- **Usage metrics:** Total, daily, weekly, and monthly OpenRouter credit consumption (in USD)
- **BYOK metrics:** External Bring-Your-Own-Key usage tracking across timeframes
- **Spending controls:** Current limit, remaining balance, and reset configuration
- **Key type indicators:** Flags for free tier, management, and provisioning keys
- **Expiration:** Optional ISO 8601 timestamp for key validity
- **Rate limiting:** Legacy rate limit data (returns -1)

## Error Responses

- **401:** Unauthorized - Authentication required or invalid credentials
- **500:** Internal server error

## SDK Examples

The documentation provides implementation examples in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all following the pattern of sending a GET request with proper Authorization headers.
