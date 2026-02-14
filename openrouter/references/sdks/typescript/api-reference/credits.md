# Credits - TypeScript SDK

## Overview

The Credits API provides endpoints for managing account credits within the OpenRouter TypeScript SDK, which is currently in beta.

## Available Operations

- **getCredits** - Retrieve remaining credits
- **createCoinbaseCharge** - Generate a Coinbase charge for cryptocurrency payments

## getCredits

Retrieve total credits purchased and consumed by the authenticated user. This requires a provisioning API key.

### Usage Example

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.credits.getCredits();
  console.log(result);
}

run();
```

### Standalone Implementation

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { creditsGetCredits } from "@openrouter/sdk/funcs/creditsGetCredits.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await creditsGetCredits(openRouter);
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("creditsGetCredits failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options` | RequestOptions | No | HTTP request configuration |
| `options.fetchOptions` | RequestInit | No | Additional HTTP headers and settings |
| `options.retries` | RetryConfig | No | Automatic retry configuration |

### Response

Returns `Promise<GetCreditsResponse>`

### Error Handling

| Error | Status | Format |
|-------|--------|--------|
| Unauthorized | 401 | application/json |
| Forbidden | 403 | application/json |
| Server Error | 500 | application/json |
| Default | 4XX, 5XX | */* |

## createCoinbaseCharge

Generate a Coinbase charge for cryptocurrency-based payments.

### Usage Example

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter();

async function run() {
  const result = await openRouter.credits.createCoinbaseCharge({
    bearer: process.env["OPENROUTER_BEARER"] ?? "",
  }, {
    amount: 100,
    sender: "0x1234567890123456789012345678901234567890",
    chainId: 1,
  });

  console.log(result);
}

run();
```

### Standalone Implementation

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { creditsCreateCoinbaseCharge } from "@openrouter/sdk/funcs/creditsCreateCoinbaseCharge.js";

const openRouter = new OpenRouterCore();

async function run() {
  const res = await creditsCreateCoinbaseCharge(openRouter, {
    bearer: process.env["OPENROUTER_BEARER"] ?? "",
  }, {
    amount: 100,
    sender: "0x1234567890123456789012345678901234567890",
    chainId: 1,
  });
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("creditsCreateCoinbaseCharge failed:", res.error);
  }
}

run();
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request` | CreateChargeRequest | Yes | Charge request data |
| `security` | CreateCoinbaseChargeSecurity | Yes | Authentication credentials |
| `options` | RequestOptions | No | HTTP request configuration |
| `options.fetchOptions` | RequestInit | No | Additional HTTP settings |
| `options.retries` | RetryConfig | No | Retry policy |

### Response

Returns `Promise<CreateCoinbaseChargeResponse>`

### Error Handling

| Error | Status | Format |
|-------|--------|--------|
| Bad Request | 400 | application/json |
| Unauthorized | 401 | application/json |
| Rate Limited | 429 | application/json |
| Server Error | 500 | application/json |
| Default | 4XX, 5XX | */* |
