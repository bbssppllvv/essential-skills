# Exchange Authorization Code for API Key

## Endpoint Overview

The OpenRouter API provides a POST endpoint at `https://openrouter.ai/api/v1/auth/keys` that allows developers to "exchange an authorization code from the PKCE flow for a user-controlled API key."

## Request Details

**Method:** POST
**Content-Type:** application/json

### Required Parameters

The request body must include:
- **code** (string): The authorization code received from OAuth redirect
- **Authorization** header (string): API key as bearer token

### Optional Parameters

- **code_verifier** (string): The code verifier if code_challenge was used in the authorization request
- **code_challenge_method** (string): Either "S256" or "plain" - indicates the method used to generate the code challenge

## Response

A successful 200 response returns:
- **key** (string): The API key for OpenRouter requests
- **user_id** (string or null): User ID associated with the API key

## Error Responses

- **400**: Bad Request - Invalid request parameters or malformed input
- **403**: Forbidden - Insufficient permissions despite successful authentication
- **500**: Internal Server Error - Unexpected server error

## SDK Code Examples

Complete implementations are provided for Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all following the same request structure with bearer token authentication and PKCE parameters.
