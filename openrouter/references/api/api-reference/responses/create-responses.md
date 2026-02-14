# Create a response

`POST https://openrouter.ai/api/v1/responses`

Creates a streaming or non-streaming response using OpenResponses API format.

## Authorization

```
Authorization: Bearer <token>
```

API key as bearer token in Authorization header.

## Request Body

```json
{
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": "Hello, how are you?"
    }
  ],
  "tools": [
    {
      "type": "function",
      "name": "get_current_weather",
      "description": "Get the current weather in a given location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string"
          }
        }
      }
    }
  ],
  "model": "anthropic/claude-4.5-sonnet-20250929",
  "temperature": 0.7,
  "top_p": 0.9
}
```

### Request Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `input` | string \| array | Conversation history or prompt content. Can be a string or array of input items. |
| `instructions` | string \| null | System instructions for the model |
| `metadata` | object | Additional metadata as key-value string pairs |
| `tools` | array | Array of tool definitions |
| `tool_choice` | string \| object | Tool choice strategy: `"auto"`, `"none"`, `"required"`, or specific function |
| `parallel_tool_calls` | boolean \| null | Whether to allow parallel tool calls |
| `model` | string | Model identifier (required) |
| `models` | array | Array of model identifiers |
| `text` | object | Response text format configuration |
| `reasoning` | object | Reasoning configuration |
| `max_output_tokens` | number \| null | Maximum response length |
| `temperature` | number \| null | Controls response randomness |
| `top_p` | number \| null | Nucleus sampling parameter |
| `top_logprobs` | integer \| null | Number of top logprobs to return |
| `max_tool_calls` | integer \| null | Maximum number of tool calls |
| `presence_penalty` | number \| null | Penalizes repeated tokens |
| `frequency_penalty` | number \| null | Frequency-based penalty |
| `top_k` | number | Top-k sampling parameter |
| `image_config` | object | Provider-specific image configuration options |
| `modalities` | array | Output modalities: `"text"` and/or `"image"` |
| `prompt_cache_key` | string \| null | Prompt cache key |
| `previous_response_id` | string \| null | Previous response ID for multi-turn |
| `prompt` | object | Prompt template with `id` and optional `variables` |
| `include` | array \| null | Includable fields: `"file_search_call.results"`, `"message.input_image.image_url"`, `"computer_call_output.output.image_url"`, `"reasoning.encrypted_content"`, `"code_interpreter_call.outputs"` |
| `background` | boolean \| null | Background processing |
| `safety_identifier` | string \| null | Safety identifier |
| `store` | boolean | Store setting (false) |
| `service_tier` | string | Service tier: `"auto"` (default) |
| `truncation` | object | Truncation settings |
| `stream` | boolean | Enable streaming responses (default: false) |
| `provider` | object \| null | Provider routing preferences |
| `plugins` | array | Plugins to enable for this request |
| `user` | string | Unique identifier for your end-user (max 128 characters) |
| `session_id` | string | Unique identifier for grouping related requests (max 128 characters) |
| `trace` | object | Metadata for observability and tracing |

### Input Content Types

Messages can include:

- `input_text` - Plain text content
- `input_image` - Images with detail level (`auto`, `high`, `low`)
- `input_file` - File content with `file_id`, `file_data`, `filename`, or `file_url`
- `input_audio` - Audio in `mp3` or `wav` format
- `input_video` - Video from URL or base64

### Input Message Roles

- `user` - User message
- `system` - System message
- `assistant` - Assistant message
- `developer` - Developer message

### Tools Configuration

```json
{
  "tools": [
    {
      "type": "function",
      "name": "function_name",
      "description": "What the function does",
      "parameters": {
        "type": "object",
        "properties": {}
      }
    }
  ],
  "tool_choice": "auto"
}
```

### Reasoning Configuration

```json
{
  "reasoning": {
    "effort": "high",
    "summary": "auto",
    "max_tokens": 1000,
    "enabled": true
  }
}
```

Effort levels: `"xhigh"`, `"high"`, `"medium"`, `"low"`, `"minimal"`, `"none"`

Summary verbosity: `"auto"`, `"concise"`, `"detailed"`

### Response Text Format

```json
{
  "text": {
    "format": {
      "type": "json_object"
    },
    "verbosity": "medium"
  }
}
```

Format types: `"text"`, `"json_object"`, `"json_schema"`

Verbosity: `"high"`, `"low"`, `"medium"`

### Provider Configuration

```json
{
  "provider": {
    "allow_fallbacks": true,
    "require_parameters": false,
    "data_collection": "deny",
    "zdr": false,
    "enforce_distillable_text": false,
    "order": ["Anthropic", "OpenAI"],
    "only": ["Anthropic"],
    "ignore": ["FakeProvider"],
    "quantizations": ["fp16", "bf16"],
    "sort": "price",
    "max_price": {
      "prompt": "0.50",
      "completion": "1.00"
    },
    "preferred_min_throughput": 50.0,
    "preferred_max_latency": 2.0
  }
}
```

**Provider parameters:**

| Parameter | Type | Description |
| --- | --- | --- |
| `allow_fallbacks` | boolean \| null | Whether to allow backup providers to serve requests. Default: true. |
| `require_parameters` | boolean \| null | Whether to filter providers to only those supporting your parameters. |
| `data_collection` | string | `"deny"` or `"allow"` |
| `zdr` | boolean \| null | Restrict to Zero Data Retention endpoints only. |
| `enforce_distillable_text` | boolean \| null | Restrict to models that allow text distillation. |
| `order` | array \| null | Ordered list of provider slugs for routing preference. |
| `only` | array \| null | List of provider slugs to allow. |
| `ignore` | array \| null | List of provider slugs to ignore. |
| `quantizations` | array \| null | Filter by quantization levels: `int4`, `int8`, `fp4`, `fp6`, `fp8`, `fp16`, `bf16`, `fp32`, `unknown` |
| `sort` | string \| object | Sorting strategy: `"price"`, `"throughput"`, `"latency"` |
| `max_price` | object | Maximum price per million tokens for prompt and completion. |
| `preferred_min_throughput` | number \| object | Minimum throughput (tokens/sec) or percentile cutoffs. |
| `preferred_max_latency` | number \| object | Maximum latency (seconds) or percentile cutoffs. |

### Plugins Configuration

```json
{
  "plugins": [
    {
      "id": "auto-router",
      "enabled": true,
      "allowed_models": ["anthropic/*"]
    },
    {
      "id": "web",
      "enabled": true,
      "max_results": 5,
      "search_prompt": "search query",
      "engine": "native"
    },
    {
      "id": "file-parser",
      "enabled": true,
      "pdf": {
        "engine": "mistral-ocr"
      }
    },
    {
      "id": "response-healing",
      "enabled": true
    },
    {
      "id": "moderation"
    }
  ]
}
```

**Plugin IDs:**

| Plugin ID | Description |
| --- | --- |
| `auto-router` | Automatic model routing. Supports `allowed_models` patterns with wildcards. |
| `moderation` | Content moderation |
| `web` | Web search. Options: `max_results`, `search_prompt`, `engine` (`"native"` or `"exa"`) |
| `file-parser` | File parsing. PDF engine options: `"mistral-ocr"`, `"pdf-text"`, `"native"` |
| `response-healing` | Response healing for malformed outputs |

### Trace Configuration

```json
{
  "trace": {
    "trace_id": "string",
    "trace_name": "string",
    "span_name": "string",
    "generation_name": "string",
    "parent_span_id": "string"
  }
}
```

Known keys (`trace_id`, `trace_name`, `span_name`, `generation_name`, `parent_span_id`) have special handling. Additional keys are passed through as custom metadata to configured broadcast destinations.

## Response (200)

Successful response returns an `OpenResponsesNonStreamingResponse`:

```json
{
  "id": "response_id",
  "object": "response",
  "created_at": 1234567890,
  "model": "anthropic/claude-4.5-sonnet-20250929",
  "status": "completed",
  "completed_at": 1234567900,
  "output": [
    {
      "id": "msg_id",
      "role": "assistant",
      "type": "message",
      "status": "completed",
      "content": [
        {
          "type": "output_text",
          "text": "Response content"
        }
      ]
    }
  ],
  "usage": {
    "input_tokens": 100,
    "input_tokens_details": {
      "cached_tokens": 0
    },
    "output_tokens": 50,
    "output_tokens_details": {
      "reasoning_tokens": 0
    },
    "total_tokens": 150,
    "cost": 0.001,
    "is_byok": false,
    "cost_details": {
      "upstream_inference_cost": 0.001,
      "upstream_inference_input_cost": 0.0005,
      "upstream_inference_output_cost": 0.0005
    }
  }
}
```

### Response Status Values

| Status | Description |
| --- | --- |
| `completed` | Fully processed |
| `incomplete` | Stopped early |
| `in_progress` | Still processing |
| `failed` | Error occurred |
| `cancelled` | User cancelled |
| `queued` | Awaiting processing |

### Output Content Types

Responses may contain:

- `output_text` - Text with optional annotations (file citations, URL citations, file paths) and logprobs
- `refusal` - Refusal content
- `function_call` - Tool invocation requests with `call_id`, `name`, `arguments`
- `web_search_call` - Web search operations
- `file_search_call` - File search operations with queries
- `image_generation_call` - Image generation requests
- `reasoning` - Extended thinking output with content, summary, optional encrypted_content and signature

### Reasoning Format Values

- `unknown`
- `openai-responses-v1`
- `azure-openai-responses-v1`
- `xai-responses-v1`
- `anthropic-claude-v1`
- `google-gemini-v1`

### Incomplete Details

When status is `incomplete`, the `incomplete_details` field indicates the reason:

- `max_output_tokens` - Hit output token limit
- `content_filter` - Content filter triggered

## Error Responses

| Status | Description |
| --- | --- |
| 400 | Bad Request - Invalid request parameters or malformed input |
| 401 | Unauthorized - Authentication required or invalid credentials |
| 402 | Payment Required - Insufficient credits or quota to complete request |
| 404 | Not Found - Resource does not exist |
| 408 | Request Timeout - Operation exceeded time limit |
| 413 | Payload Too Large - Request payload exceeds size limits |
| 422 | Unprocessable Entity - Semantic validation failure |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Unexpected server error |
| 502 | Bad Gateway - Provider/upstream API failure |
| 503 | Service Unavailable - Service temporarily unavailable |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/responses"

payload = {
    "input": [
        {
            "type": "message",
            "role": "user",
            "content": "Hello, how are you?"
        }
    ],
    "tools": [
        {
            "type": "function",
            "name": "get_current_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": { "location": { "type": "string" } }
            }
        }
    ],
    "model": "anthropic/claude-4.5-sonnet-20250929",
    "temperature": 0.7,
    "top_p": 0.9
}
headers = {
    "Authorization": "Bearer <token>",
    "Content-Type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/responses';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"input":[{"type":"message","role":"user","content":"Hello, how are you?"}],"tools":[{"type":"function","name":"get_current_weather","description":"Get the current weather in a given location","parameters":{"type":"object","properties":{"location":{"type":"string"}}}}],"model":"anthropic/claude-4.5-sonnet-20250929","temperature":0.7,"top_p":0.9}'
};

try {
  const response = await fetch(url, options);
  const data = await response.json();
  console.log(data);
} catch (error) {
  console.error(error);
}
```

### Go

```go
package main

import (
	"fmt"
	"strings"
	"net/http"
	"io"
)

func main() {

	url := "https://openrouter.ai/api/v1/responses"

	payload := strings.NewReader("{\n  \"input\": [\n    {\n      \"type\": \"message\",\n      \"role\": \"user\",\n      \"content\": \"Hello, how are you?\"\n    }\n  ],\n  \"tools\": [\n    {\n      \"type\": \"function\",\n      \"name\": \"get_current_weather\",\n      \"description\": \"Get the current weather in a given location\",\n      \"parameters\": {\n        \"type\": \"object\",\n        \"properties\": {\n          \"location\": {\n            \"type\": \"string\"\n          }\n        }\n      }\n    }\n  ],\n  \"model\": \"anthropic/claude-4.5-sonnet-20250929\",\n  \"temperature\": 0.7,\n  \"top_p\": 0.9\n}")

	req, _ := http.NewRequest("POST", url, payload)

	req.Header.Add("Authorization", "Bearer <token>")
	req.Header.Add("Content-Type", "application/json")

	res, _ := http.DefaultClient.Do(req)

	defer res.Body.Close()
	body, _ := io.ReadAll(res.Body)

	fmt.Println(res)
	fmt.Println(string(body))

}
```

### Ruby

```ruby
require 'uri'
require 'net/http'

url = URI("https://openrouter.ai/api/v1/responses")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"input\": [\n    {\n      \"type\": \"message\",\n      \"role\": \"user\",\n      \"content\": \"Hello, how are you?\"\n    }\n  ],\n  \"tools\": [\n    {\n      \"type\": \"function\",\n      \"name\": \"get_current_weather\",\n      \"description\": \"Get the current weather in a given location\",\n      \"parameters\": {\n        \"type\": \"object\",\n        \"properties\": {\n          \"location\": {\n            \"type\": \"string\"\n          }\n        }\n      }\n    }\n  ],\n  \"model\": \"anthropic/claude-4.5-sonnet-20250929\",\n  \"temperature\": 0.7,\n  \"top_p\": 0.9\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/responses")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"input\": [\n    {\n      \"type\": \"message\",\n      \"role\": \"user\",\n      \"content\": \"Hello, how are you?\"\n    }\n  ],\n  \"tools\": [\n    {\n      \"type\": \"function\",\n      \"name\": \"get_current_weather\",\n      \"description\": \"Get the current weather in a given location\",\n      \"parameters\": {\n        \"type\": \"object\",\n        \"properties\": {\n          \"location\": {\n            \"type\": \"string\"\n          }\n        }\n      }\n    }\n  ],\n  \"model\": \"anthropic/claude-4.5-sonnet-20250929\",\n  \"temperature\": 0.7,\n  \"top_p\": 0.9\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/responses', [
  'body' => '{
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": "Hello, how are you?"
    }
  ],
  "tools": [
    {
      "type": "function",
      "name": "get_current_weather",
      "description": "Get the current weather in a given location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string"
          }
        }
      }
    }
  ],
  "model": "anthropic/claude-4.5-sonnet-20250929",
  "temperature": 0.7,
  "top_p": 0.9
}',
  'headers' => [
    'Authorization' => 'Bearer <token>',
    'Content-Type' => 'application/json',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/responses");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"input\": [\n    {\n      \"type\": \"message\",\n      \"role\": \"user\",\n      \"content\": \"Hello, how are you?\"\n    }\n  ],\n  \"tools\": [\n    {\n      \"type\": \"function\",\n      \"name\": \"get_current_weather\",\n      \"description\": \"Get the current weather in a given location\",\n      \"parameters\": {\n        \"type\": \"object\",\n        \"properties\": {\n          \"location\": {\n            \"type\": \"string\"\n          }\n        }\n      }\n    }\n  ],\n  \"model\": \"anthropic/claude-4.5-sonnet-20250929\",\n  \"temperature\": 0.7,\n  \"top_p\": 0.9\n}", ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = [
  "Authorization": "Bearer <token>",
  "Content-Type": "application/json"
]
let parameters = [
  "input": [
    [
      "type": "message",
      "role": "user",
      "content": "Hello, how are you?"
    ]
  ],
  "tools": [
    [
      "type": "function",
      "name": "get_current_weather",
      "description": "Get the current weather in a given location",
      "parameters": [
        "type": "object",
        "properties": ["location": ["type": "string"]]
      ]
    ]
  ],
  "model": "anthropic/claude-4.5-sonnet-20250929",
  "temperature": 0.7,
  "top_p": 0.9
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/responses")! as URL,
                                        cachePolicy: .useProtocolCachePolicy,
                                    timeoutInterval: 10.0)
request.httpMethod = "POST"
request.allHTTPHeaderFields = headers
request.httpBody = postData as Data

let session = URLSession.shared
let dataTask = session.dataTask(with: request as URLRequest, completionHandler: { (data, response, error) -> Void in
  if (error != nil) {
    print(error as Any)
  } else {
    let httpResponse = response as? HTTPURLResponse
    print(httpResponse)
  }
})

dataTask.resume()
```
