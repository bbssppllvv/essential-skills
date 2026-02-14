# OAuth - TypeScript SDK Documentation

## Overview

This documentation covers OAuth authentication endpoints within the OpenRouter TypeScript SDK, currently in beta phase with issues reported on GitHub.

## Available Operations

1. **exchangeAuthCodeForAPIKey** - Exchanges an authorization code from the PKCE flow for a user-controlled API key
2. **createAuthCode** - Creates an authorization code for the PKCE flow to generate a user-controlled API key

## exchangeAuthCodeForAPIKey Method

### Purpose
Converts authorization codes obtained through PKCE flow into user-controlled API keys.

### Basic Implementation
```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.oAuth.exchangeAuthCodeForAPIKey({
    code: "auth_code_abc123def456",
  });
  console.log(result);
}
```

### Parameters
- **request**: Exchange request object (required)
- **options**: HTTP request configuration (optional)
- **options.fetchOptions**: Browser Request/Response API options (optional)
- **options.retries**: Retry configuration settings (optional)

### Possible Error Responses
- 400 Bad Request
- 403 Forbidden
- 500 Internal Server Error
- 4XX/5XX General OpenRouter errors

## createAuthCode Method

### Purpose
Initiates PKCE flow by generating authorization codes for user-controlled API key creation.

### Basic Implementation
```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.oAuth.createAuthCode({
    callbackUrl: "https://myapp.com/auth/callback",
  });
  console.log(result);
}
```

### Parameters
- **request**: Auth code creation request with callback URL (required)
- **options**: HTTP configuration options (optional)
- **options.fetchOptions**: Standard fetch request options (optional)
- **options.retries**: Automatic retry settings (optional)

### Possible Error Responses
- 400 Bad Request
- 401 Unauthorized
- 500 Internal Server Error
- 4XX/5XX General OpenRouter errors
