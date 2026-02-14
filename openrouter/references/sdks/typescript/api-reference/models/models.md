# Models - TypeScript SDK Documentation

## Overview

The Models endpoints provide access to model information through the OpenRouter TypeScript SDK, which is currently in beta.

### Available Operations

- **count** - Retrieve the total number of available models
- **list** - Display all models with their properties
- **listForUser** - Access models filtered by user preferences, privacy settings, and guardrails

## count

Fetches the total count of available models.

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.models.count();
  console.log(result);
}

run();
```

**Standalone function approach:**

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { modelsCount } from "@openrouter/sdk/funcs/modelsCount.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await modelsCount(openRouter);
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("modelsCount failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options` | RequestOptions | No | HTTP request configuration |
| `options.fetchOptions` | RequestInit | No | Underlying HTTP request options |
| `options.retries` | RetryConfig | No | Retry configuration for failed requests |

### Response & Errors

- **Response:** `Promise<models.ModelsCountResponse>`
- **Errors:** 500 (InternalServerResponseError), 4XX/5XX (OpenRouterDefaultError)

---

## list

Returns all available models and their associated properties.

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.models.list();
  console.log(result);
}

run();
```

**Standalone function:**

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { modelsList } from "@openrouter/sdk/funcs/modelsList.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await modelsList(openRouter);
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("modelsList failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request` | GetModelsRequest | Yes | The request object |
| `options` | RequestOptions | No | HTTP configuration |
| `options.fetchOptions` | RequestInit | No | Underlying HTTP options |
| `options.retries` | RetryConfig | No | Retry behavior |

### Response & Errors

- **Response:** `Promise<models.ModelsListResponse>`
- **Errors:** 400 (BadRequestResponseError), 500 (InternalServerResponseError), 4XX/5XX (OpenRouterDefaultError)

---

## listForUser

Provides a filtered model list based on user provider preferences, privacy settings, and guardrails. EU-specific routing applies when accessed through `eu.openrouter.ai/api/v1/...`.

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter();

async function run() {
  const result = await openRouter.models.listForUser({
    bearer: process.env["OPENROUTER_BEARER"] ?? "",
  });
  console.log(result);
}

run();
```

**Standalone function:**

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { modelsListForUser } from "@openrouter/sdk/funcs/modelsListForUser.js";

const openRouter = new OpenRouterCore();

async function run() {
  const res = await modelsListForUser(openRouter, {
    bearer: process.env["OPENROUTER_BEARER"] ?? "",
  });
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("modelsListForUser failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `security` | ListModelsUserSecurity | Yes | Security authentication |
| `options` | RequestOptions | No | HTTP options |
| `options.fetchOptions` | RequestInit | No | HTTP request options |
| `options.retries` | RetryConfig | No | Retry configuration |

### Response & Errors

- **Response:** `Promise<models.ModelsListResponse>`
- **Errors:** 401 (UnauthorizedResponseError), 404 (NotFoundResponseError), 500 (InternalServerResponseError), 4XX/5XX (OpenRouterDefaultError)
