# OpenRouter TypeScript SDK - Chat Method Reference

## Overview

The Chat method enables developers to create chat completions through the OpenRouter API, supporting both streaming and non-streaming modes.

## Available Operations

- **send** - Create a chat completion

## send Method

### Purpose
Sends requests for model responses in chat conversations with flexible streaming options.

### Basic Usage Example

```typescript
import { OpenRouter } from "@openrouter/sdk";

const openRouter = new OpenRouter({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const result = await openRouter.chat.send({
    messages: [],
  });

  console.log(result);
}

run();
```

### Standalone Function Approach

```typescript
import { OpenRouterCore } from "@openrouter/sdk/core.js";
import { chatSend } from "@openrouter/sdk/funcs/chatSend.js";

const openRouter = new OpenRouterCore({
  apiKey: process.env["OPENROUTER_API_KEY"] ?? "",
});

async function run() {
  const res = await chatSend(openRouter, {
    messages: [],
  });
  if (res.ok) {
    const { value: result } = res;
    console.log(result);
  } else {
    console.log("chatSend failed:", res.error);
  }
}

run();
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `request` | ChatGenerationParams | Yes | The request object for the API call |
| `options` | RequestOptions | No | HTTP request configuration options |
| `options.fetchOptions` | RequestInit | No | Underlying HTTP request settings (excluding method/body) |
| `options.retries` | RetryConfig | No | Configuration for automatic retry behavior |

## Response

Returns: `Promise<SendChatCompletionRequestResponse>`

## Error Handling

| Error Type | Status Codes | Content Type |
|------------|--------------|--------------|
| ChatError | 400, 401, 429, 500 | application/json |
| OpenRouterDefaultError | 4XX, 5XX | */* |

---

**Note:** The TypeScript SDK and documentation are currently in beta. Issues can be reported on GitHub.
