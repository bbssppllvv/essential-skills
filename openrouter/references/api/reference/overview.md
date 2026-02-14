# API Reference

An overview of OpenRouter's API

## Overview

OpenRouter's request and response schemas closely mirror the OpenAI Chat API with minor differences. The platform normalizes the schema across models and providers so you only need to learn one.

## OpenAPI Specification

Complete API documentation is available via OpenAPI specification:

- **YAML:** https://openrouter.ai/openapi.yaml
- **JSON:** https://openrouter.ai/openapi.json

These can be used with Swagger UI, Postman, or OpenAPI-compatible code generators.

---

## Requests

### Completions Request Format

POST requests to `/api/v1/chat/completions` use the following schema:

```typescript
type Request = {
  // Either "messages" or "prompt" is required
  messages?: Message[];
  prompt?: string;

  // If "model" is unspecified, uses the user's default
  model?: string; // See "Supported Models" section

  // Allows to force the model to produce specific output format.
  // See "Structured Outputs" section below and models page for which models support it.
  response_format?: ResponseFormat;

  stop?: string | string[];
  stream?: boolean; // Enable streaming

  // Plugins to extend model capabilities (web search, PDF parsing, response healing)
  // See "Plugins" section: openrouter.ai/docs/guides/features/plugins
  plugins?: Plugin[];

  // See LLM Parameters (openrouter.ai/docs/api/reference/parameters)
  max_tokens?: number; // Range: [1, context_length)
  temperature?: number; // Range: [0, 2]

  // Tool calling
  // Will be passed down as-is for providers implementing OpenAI's interface.
  // For providers with custom interfaces, we transform and map the properties.
  // Otherwise, we transform the tools into a YAML template. The model responds with an assistant message.
  // See models supporting tool calling: openrouter.ai/models?supported_parameters=tools
  tools?: Tool[];
  tool_choice?: ToolChoice;

  // Advanced optional parameters
  seed?: number; // Integer only
  top_p?: number; // Range: (0, 1]
  top_k?: number; // Range: [1, Infinity) Not available for OpenAI models
  frequency_penalty?: number; // Range: [-2, 2]
  presence_penalty?: number; // Range: [-2, 2]
  repetition_penalty?: number; // Range: (0, 2]
  logit_bias?: { [key: number]: number };
  top_logprobs: number; // Integer only
  min_p?: number; // Range: [0, 1]
  top_a?: number; // Range: [0, 1]

  // Reduce latency by providing the model with a predicted output
  // https://platform.openai.com/docs/guides/latency-optimization#use-predicted-outputs
  prediction?: { type: 'content'; content: string };

  // OpenRouter-only parameters
  // See "Prompt Transforms" section: openrouter.ai/docs/guides/features/message-transforms
  transforms?: string[];
  // See "Model Routing" section: openrouter.ai/docs/guides/features/model-routing
  models?: string[];
  route?: 'fallback';
  // See "Provider Routing" section: openrouter.ai/docs/guides/routing/provider-selection
  provider?: ProviderPreferences;
  user?: string; // A stable identifier for your end-users. Used to help detect and prevent abuse.

  // Debug options (streaming only)
  debug?: {
    echo_upstream_body?: boolean; // If true, returns the transformed request body sent to the provider
  };
};

// Subtypes:

type TextContent = {
  type: 'text';
  text: string;
};

type ImageContentPart = {
  type: 'image_url';
  image_url: {
    url: string; // URL or base64 encoded image data
    detail?: string; // Optional, defaults to "auto"
  };
};

type ContentPart = TextContent | ImageContentPart;

type Message =
  | {
      role: 'user' | 'assistant' | 'system';
      // ContentParts are only for the "user" role:
      content: string | ContentPart[];
      // If "name" is included, it will be prepended like this
      // for non-OpenAI models: `{name}: {content}`
      name?: string;
    }
  | {
      role: 'tool';
      content: string;
      tool_call_id: string;
      name?: string;
    };

type FunctionDescription = {
  description?: string;
  name: string;
  parameters: object; // JSON Schema object
};

type Tool = {
  type: 'function';
  function: FunctionDescription;
};

type ToolChoice =
  | 'none'
  | 'auto'
  | {
      type: 'function';
      function: {
        name: string;
      };
    };

// Response format for structured outputs
type ResponseFormat =
  | { type: 'json_object' }
  | {
      type: 'json_schema';
      json_schema: {
        name: string;
        strict?: boolean;
        schema: object; // JSON Schema object
      };
    };

// Plugin configuration
type Plugin = {
  id: string; // 'web', 'file-parser', 'response-healing'
  enabled?: boolean;
  // Additional plugin-specific options
  [key: string]: unknown;
};
```

### Structured Outputs

The `response_format` parameter enforces structured JSON responses:

- `{ type: 'json_object' }`: Basic JSON mode - model returns valid JSON
- `{ type: 'json_schema', json_schema: { ... } }`: Strict schema mode - model returns JSON matching exact schema

See [Structured Outputs](/docs/guides/features/structured-outputs) for detailed usage and examples, and the [models page](https://openrouter.ai/models?supported_parameters=structured_outputs) for supported models.

### Plugins

OpenRouter plugins extend capabilities with web search, PDF processing, and response healing. Enable by adding a `plugins` array:

```json
{
  "plugins": [
    { "id": "web" },
    { "id": "response-healing" }
  ]
}
```

Available plugins: `web` (real-time web search), `file-parser` (PDF processing), and `response-healing` (automatic JSON repair). See [Plugins](/docs/guides/features/plugins) for configuration options.

### Headers

Optional headers to identify your app:

- `HTTP-Referer`: Identifies your app on openrouter.ai
- `X-Title`: Sets/modifies your app's title

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>', // Optional. Site URL for rankings on openrouter.ai.
    'X-Title': '<YOUR_SITE_NAME>', // Optional. Site title for rankings on openrouter.ai.
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      {
        role: 'user',
        content: 'What is the meaning of life?',
      },
    ],
  }),
});
```

### Model Routing

If the `model` parameter is omitted, the user or payer's default is used. Otherwise, select a value from [supported models](/models) or [API](/api/v1/models) with the organization prefix. OpenRouter selects the least expensive and best GPUs available, falling back to other providers or GPUs on 5xx responses or rate limiting.

### Streaming

Server-Sent Events (SSE) are supported for all models. Send `stream: true` in your request body. The SSE stream occasionally contains "comment" payloads that should be ignored.

### Non-standard Parameters

If the chosen model doesn't support a request parameter (such as `logit_bias` for non-OpenAI models or `top_k` for OpenAI), the parameter is ignored and forwarded to the underlying model API.

### Assistant Prefill

Models can complete partial responses by including a message with `role: "assistant"` at the end of the `messages` array:

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      { role: 'user', content: 'What is the meaning of life?' },
      { role: 'assistant', content: "I'm not sure, but my best guess is" },
    ],
  }),
});
```

---

## Responses

### CompletionsResponse Format

OpenRouter normalizes the schema across models and providers to comply with the [OpenAI Chat API](https://platform.openai.com/docs/api-reference/chat). The `choices` array is always returned, even for a single completion. Each choice contains a `delta` property for streams or a `message` property otherwise.

```typescript
type Response = {
  id: string;
  // Depending on whether you set "stream" to "true" and
  // whether you passed in "messages" or a "prompt", you
  // will get a different output shape
  choices: (NonStreamingChoice | StreamingChoice | NonChatChoice)[];
  created: number; // Unix timestamp
  model: string;
  object: 'chat.completion' | 'chat.completion.chunk';

  system_fingerprint?: string; // Only present if the provider supports it

  // Usage data is always returned for non-streaming.
  // When streaming, usage is returned exactly once in the final chunk
  // before the [DONE] message, with an empty choices array.
  usage?: ResponseUsage;
};
```

### ResponseUsage

```typescript
type ResponseUsage = {
  /** Including images, input audio, and tools if any */
  prompt_tokens: number;
  /** The tokens generated */
  completion_tokens: number;
  /** Sum of the above two fields */
  total_tokens: number;

  /** Breakdown of prompt tokens (optional) */
  prompt_tokens_details?: {
    cached_tokens: number;        // Tokens cached by the endpoint
    cache_write_tokens?: number;  // Tokens written to cache (models with explicit caching)
    audio_tokens?: number;        // Tokens used for input audio
    video_tokens?: number;        // Tokens used for input video
  };

  /** Breakdown of completion tokens (optional) */
  completion_tokens_details?: {
    reasoning_tokens?: number;    // Tokens generated for reasoning
    image_tokens?: number;        // Tokens generated for image output
  };

  /** Cost in credits (optional) */
  cost?: number;
  /** Whether request used Bring Your Own Key */
  is_byok?: boolean;
  /** Detailed cost breakdown (optional) */
  cost_details?: {
    upstream_inference_cost?: number;             // Only shown for BYOK requests
    upstream_inference_prompt_cost: number;
    upstream_inference_completions_cost: number;
  };

  /** Server-side tool usage (optional) */
  server_tool_use?: {
    web_search_requests?: number;
  };
};
```

### Choice Subtypes

```typescript
type NonChatChoice = {
  finish_reason: string | null;
  text: string;
  error?: ErrorResponse;
};

type NonStreamingChoice = {
  finish_reason: string | null;
  native_finish_reason: string | null;
  message: {
    content: string | null;
    role: string;
    tool_calls?: ToolCall[];
  };
  error?: ErrorResponse;
};

type StreamingChoice = {
  finish_reason: string | null;
  native_finish_reason: string | null;
  delta: {
    content: string | null;
    role?: string;
    tool_calls?: ToolCall[];
  };
  error?: ErrorResponse;
};

type ErrorResponse = {
  code: number; // See "Error Handling" section
  message: string;
  metadata?: Record<string, unknown>; // Contains additional error information such as provider details, the raw error message, etc.
};

type ToolCall = {
  id: string;
  type: 'function';
  function: FunctionCall;
};
```

### Example Response

```json
{
  "id": "gen-xxxxxxxxxxxxxx",
  "choices": [
    {
      "finish_reason": "stop",
      "native_finish_reason": "stop",
      "message": {
        "role": "assistant",
        "content": "Hello there!"
      }
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 4,
    "total_tokens": 14,
    "prompt_tokens_details": {
      "cached_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0
    },
    "cost": 0.00014
  },
  "model": "openai/gpt-3.5-turbo"
}
```

### Finish Reason

OpenRouter normalizes `finish_reason` to: `tool_calls`, `stop`, `length`, `content_filter`, `error`. Some models may have additional finish reasons. The raw finish_reason is available via `native_finish_reason`.

### Querying Cost and Stats

Token counts are calculated using the model's native tokenizer. Use the returned `id` to query generation stats via `/api/v1/generation` endpoint for auditing or asynchronous stat retrieval:

```typescript
const generation = await fetch(
  'https://openrouter.ai/api/v1/generation?id=$GENERATION_ID',
  { headers },
);

const stats = await generation.json();
```

See the [Generation](/docs/api-reference/get-a-generation) API reference for full response shape. Token counts are also available in the `usage` field for non-streaming completions.
