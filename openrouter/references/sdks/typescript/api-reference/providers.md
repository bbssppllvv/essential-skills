# Providers - TypeScript SDK

## Overview

This documentation covers provider information endpoints within the OpenRouter TypeScript SDK, which is currently in beta.

## Available Operations

- **list** - Retrieve all available providers

## List Providers

### Description

Fetches a complete list of all providers.

### Usage Example

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.providers.list();
  console.log(result);
}

run();
```

### Standalone Function Approach

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { providersList } from "@openrouter/sdk/funcs/providersList.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await providersList(openRouter);
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("providersList failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Details |
|-----------|------|----------|---------|
| `options` | RequestOptions | No | Configures HTTP request behavior |
| `options.fetchOptions` | RequestInit | No | HTTP request options (headers, etc.) |
| `options.retries` | RetryConfig | No | Retry configuration for failed requests |

### Response

Returns a Promise containing `operations.ListProvidersResponse`

### Possible Errors

| Error | Status | Format |
|-------|--------|--------|
| InternalServerResponseError | 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | \*/\* |
