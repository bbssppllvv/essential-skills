# Analytics - TypeScript SDK Documentation

## Overview

This documentation covers the Analytics and usage endpoints for the OpenRouter TypeScript SDK, which is currently in beta.

## Available Operations

- **getUserActivity** - Retrieves user activity grouped by endpoint

## getUserActivity Method

### Purpose

"Returns user activity data grouped by endpoint for the last 30 (completed) UTC days." A provisioning API key is required to use this endpoint.

### Basic Implementation

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.analytics.getUserActivity();
  console.log(result);
}

run();
```

### Standalone Function Approach

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { analyticsGetUserActivity } from "@openrouter/sdk/funcs/analyticsGetUserActivity.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await analyticsGetUserActivity(openRouter);
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("analyticsGetUserActivity failed:", res.error);
  }
}

run();
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request` | GetUserActivityRequest | Yes | The request object |
| `options` | RequestOptions | No | HTTP request configuration |
| `options.fetchOptions` | RequestInit | No | HTTP headers and other request options |
| `options.retries` | RetryConfig | No | Retry configuration for failed requests |

## Response & Error Handling

**Return Type:** `Promise<GetUserActivityResponse>`

**Possible Errors:**
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 500: Internal Server Error
- 4XX/5XX: General OpenRouter errors
