# Generations - TypeScript SDK Documentation

## Overview

This documentation covers the generation history endpoints for the OpenRouter TypeScript SDK, which is currently in beta. Issues can be reported on GitHub.

## Available Operations

- **getGeneration** - Retrieves request and usage metadata for a generation

## getGeneration Method

Retrieves metadata about a specific generation request and its usage.

### Basic Usage

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.generations.getGeneration({
    id: "<id>",
  });

  console.log(result);
}

run();
```

### Standalone Function Approach

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { generationsGetGeneration } from "@openrouter/sdk/funcs/generationsGetGeneration.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await generationsGetGeneration(openRouter, {
    id: "<id>",
  });
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("generationsGetGeneration failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request` | GetGenerationRequest | Yes | The request object containing required parameters |
| `options` | RequestOptions | No | HTTP request configuration options |
| `options.fetchOptions` | RequestInit | No | Underlying HTTP request options (excludes method/body) |
| `options.retries` | RetryConfig | No | Configuration for retrying failed requests |

### Response

Returns a Promise of type `GetGenerationResponse`

### Possible Errors

| Error Type | Status Code | Content Type |
|-----------|-------------|--------------|
| UnauthorizedResponseError | 401 | application/json |
| PaymentRequiredResponseError | 402 | application/json |
| NotFoundResponseError | 404 | application/json |
| TooManyRequestsResponseError | 429 | application/json |
| InternalServerResponseError | 500 | application/json |
| BadGatewayResponseError | 502 | application/json |
| EdgeNetworkTimeoutResponseError | 524 | application/json |
| ProviderOverloadedResponseError | 529 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |
