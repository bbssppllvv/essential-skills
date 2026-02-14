# Endpoints - TypeScript SDK Documentation

## Overview

This document provides endpoint information for the OpenRouter TypeScript SDK, which is currently in beta. Users encountering issues should report them on GitHub.

## Available Operations

The SDK supports two primary operations for managing endpoints:

1. **list** - Retrieves all endpoints available for a specific model
2. **listZdrEndpoints** - Shows the potential impact of ZDR on accessible endpoints

## list Operation

### Purpose
Lists all endpoints for a designated model using author and slug parameters.

### Implementation Example
```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.endpoints.list({
    author: "<value>",
    slug: "<value>",
  });
  console.log(result);
}
run();
```

### Standalone Function Approach
```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { endpointsList } from "@openrouter/sdk/funcs/endpointsList.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await endpointsList(openRouter, {
    author: "<value>",
    slug: "<value>",
  });
  if (res.ok) {
    console.log(res.value);
  } else {
    console.log("endpointsList failed:", res.error);
  }
}
run();
```

### Parameters

| Parameter | Type | Required | Purpose |
|-----------|------|----------|---------|
| request | ListEndpointsRequest | Yes | Request object for the operation |
| options | RequestOptions | No | HTTP configuration settings |
| options.fetchOptions | RequestInit | No | Underlying HTTP request options |
| options.retries | RetryConfig | No | HTTP retry configuration |

### Response Type
`Promise<operations.ListEndpointsResponse>`

### Possible Errors
- 404: NotFoundResponseError
- 500: InternalServerResponseError
- 4XX, 5XX: OpenRouterDefaultError

## listZdrEndpoints Operation

### Purpose
Previews how Zero Downtime Routing (ZDR) impacts endpoint availability.

### Implementation Example
```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.endpoints.listZdrEndpoints();
  console.log(result);
}
run();
```

### Standalone Function Approach
```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { endpointsListZdrEndpoints } from "@openrouter/sdk/funcs/endpointsListZdrEndpoints.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await endpointsListZdrEndpoints(openRouter);
  if (res.ok) {
    console.log(res.value);
  } else {
    console.log("endpointsListZdrEndpoints failed:", res.error);
  }
}
run();
```

### Parameters

| Parameter | Type | Required | Purpose |
|-----------|------|----------|---------|
| options | RequestOptions | No | HTTP configuration settings |
| options.fetchOptions | RequestInit | No | Underlying HTTP request options |
| options.retries | RetryConfig | No | HTTP retry configuration |

### Response Type
`Promise<operations.ListEndpointsZdrResponse>`

### Possible Errors
- 500: InternalServerResponseError
- 4XX, 5XX: OpenRouterDefaultError
