# Chat - Python SDK Documentation

## Overview

The Chat method enables model responses for given chat conversations, supporting both streaming and non-streaming modes.

### Available Operations

- **send** - Create a chat completion

## send Method

This operation sends a request for model responses in chat conversations with flexible streaming options.

### Example Usage

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY", ""),
) as open_router:

    res = open_router.chat.send(messages=[], stream=False)

    with res as event_stream:
        for event in event_stream:
            # handle event
            print(event, flush=True)
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `messages` | List[components.Message] | Yes | Chat message content |
| `provider` | OptionalNullable[components.ChatGenerationParamsProvider] | No | Route preference when multiple providers available |
| `plugins` | List[components.ChatGenerationParamsPluginUnion] | No | Plugins with settings |
| `route` | OptionalNullable[components.Route] | No | Route configuration |
| `user` | Optional[str] | No | User identifier |
| `session_id` | Optional[str] | No | A unique identifier for grouping related requests (e.g., a conversation or agent workflow) for observability. Maximum of 128 characters. |
| `model` | Optional[str] | No | Model selection |
| `models` | List[str] | No | Multiple models |
| `frequency_penalty` | OptionalNullable[float] | No | Frequency penalty |
| `logit_bias` | Dict[str, float] | No | Logit bias mapping |
| `logprobs` | OptionalNullable[bool] | No | Log probabilities |
| `top_logprobs` | OptionalNullable[float] | No | Top log probabilities |
| `max_completion_tokens` | OptionalNullable[float] | No | Maximum completion tokens |
| `max_tokens` | OptionalNullable[float] | No | Maximum tokens |
| `metadata` | Dict[str, str] | No | Custom metadata |
| `presence_penalty` | OptionalNullable[float] | No | Presence penalty |
| `reasoning` | Optional[components.Reasoning] | No | Reasoning configuration |
| `response_format` | Optional[components.ResponseFormat] | No | Response format |
| `seed` | OptionalNullable[int] | No | Random seed |
| `stop` | OptionalNullable[components.Stop] | No | Stop sequences |
| `stream` | Optional[bool] | No | Enable streaming |
| `stream_options` | OptionalNullable[components.ChatStreamOptions] | No | Stream options |
| `temperature` | OptionalNullable[float] | No | Sampling temperature |
| `tool_choice` | Optional[Any] | No | Tool selection |
| `tools` | List[components.ToolDefinitionJSON] | No | Available tools |
| `top_p` | OptionalNullable[float] | No | Top-p sampling |
| `debug` | Optional[components.Debug] | No | Debug mode |
| `image_config` | Dict[str, components.ChatGenerationParamsImageConfig] | No | Image configuration |
| `modalities` | List[components.Modality] | No | Content modalities |
| `retries` | Optional[utils.RetryConfig] | No | Retry configuration |

### Response

Returns: `operations.SendChatCompletionRequestResponse`

### Error Handling

| Error Type | Status Code | Content Type |
|-----------|-------------|--------------|
| errors.ChatError | 400, 401, 429 | application/json |
| errors.ChatError | 500 | application/json |
| errors.OpenRouterDefaultError | 4XX, 5XX | \*/\* |

**Note:** The Python SDK and documentation are currently in beta. Report issues on [GitHub](https://github.com/OpenRouterTeam/python-sdk/issues).
