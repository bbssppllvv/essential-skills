# Beta.Responses - TypeScript SDK

## Overview

The `beta.responses` endpoints provide functionality to create streaming or non-streaming responses using the OpenResponses API format.

## Available Operations

- **send** - Create a response

## send Method

Creates a streaming or non-streaming response using OpenResponses API format.

### Basic Usage

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.beta.responses.send({});
  console.log(result);
}

run();
```

### Standalone Function Usage

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { betaResponsesSend } from "@openrouter/sdk/funcs/betaResponsesSend.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await betaResponsesSend(openRouter, {});
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("betaResponsesSend failed:", res.error);
  }
}

run();
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request` | OpenResponsesRequest | Yes | Request object for the API call |
| `options` | RequestOptions | No | HTTP request configuration options |
| `options.fetchOptions` | RequestInit | No | Extra headers and fetch options (excludes method/body) |
| `options.retries` | RetryConfig | No | Retry configuration for failed requests |

## Response

Returns a Promise resolving to `CreateResponsesResponse`.

## Error Handling

Possible error responses:

| Error | Status | Type |
|-------|--------|------|
| Bad Request | 400 | application/json |
| Unauthorized | 401 | application/json |
| Payment Required | 402 | application/json |
| Not Found | 404 | application/json |
| Request Timeout | 408 | application/json |
| Payload Too Large | 413 | application/json |
| Unprocessable Entity | 422 | application/json |
| Too Many Requests | 429 | application/json |
| Internal Server Error | 500 | application/json |
| Bad Gateway | 502 | application/json |
| Service Unavailable | 503 | application/json |
| Edge Network Timeout | 524 | application/json |
| Provider Overloaded | 529 | application/json |

**Note:** The TypeScript SDK is currently in beta. Report issues on GitHub.
