# Embeddings - TypeScript SDK Documentation

## Overview

The embeddings endpoints enable text embedding requests through the OpenRouter TypeScript SDK. The API provides two main operations for working with embeddings models.

## Available Operations

- **generate** - Submit an embedding request
- **listModels** - List all embeddings models

## Generate Embeddings

### Basic Usage

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.embeddings.generate({
    input: "<value>",
    model: "Taurus",
  });

  console.log(result);
}

run();
```

### Standalone Function

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { embeddingsGenerate } from "@openrouter/sdk/funcs/embeddingsGenerate.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await embeddingsGenerate(openRouter, {
    input: "<value>",
    model: "Taurus",
  });
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("embeddingsGenerate failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request` | CreateEmbeddingsRequest | Yes | The request object for the API call |
| `options` | RequestOptions | No | HTTP request configuration options |
| `options.fetchOptions` | RequestInit | No | Additional fetch options (excluding method and body) |
| `options.retries` | RetryConfig | No | Retry configuration for failed requests |

### Response

Returns a `Promise<CreateEmbeddingsResponse>`

### Error Handling

| Error Type | Status Code |
|-----------|-------------|
| BadRequestResponseError | 400 |
| UnauthorizedResponseError | 401 |
| PaymentRequiredResponseError | 402 |
| NotFoundResponseError | 404 |
| TooManyRequestsResponseError | 429 |
| InternalServerResponseError | 500 |
| BadGatewayResponseError | 502 |
| ServiceUnavailableResponseError | 503 |
| EdgeNetworkTimeoutResponseError | 524 |
| ProviderOverloadedResponseError | 529 |
| OpenRouterDefaultError | 4XX, 5XX |

## List Embeddings Models

### Basic Usage

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.embeddings.listModels();

  console.log(result);
}

run();
```

### Standalone Function

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { embeddingsListModels } from "@openrouter/sdk/funcs/embeddingsListModels.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await embeddingsListModels(openRouter);
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("embeddingsListModels failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options` | RequestOptions | No | HTTP request configuration options |
| `options.fetchOptions` | RequestInit | No | Additional fetch options (excluding method and body) |
| `options.retries` | RetryConfig | No | Retry configuration for failed requests |

### Response

Returns a `Promise<ModelsListResponse>`

### Error Handling

| Error Type | Status Code |
|-----------|-------------|
| BadRequestResponseError | 400 |
| InternalServerResponseError | 500 |
| OpenRouterDefaultError | 4XX, 5XX |

---

**Note:** "The TypeScript SDK and docs are currently in beta."
