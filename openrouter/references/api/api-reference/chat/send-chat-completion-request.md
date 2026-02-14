# Send Chat Completion Request

Sends a request for a model response for the given chat conversation. Supports both streaming and non-streaming modes.

**Endpoint:** `POST https://openrouter.ai/api/v1/chat/completions`

**Content-Type:** `application/json`

**Authorization:** `Bearer <API_KEY>`

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/chat/completions"

payload = { "messages": [
        {
            "role": "user",
            "content": "Can you explain the theory of relativity in simple terms?"
        }
    ] }
headers = {
    "Authorization": "Bearer <token>",
    "Content-Type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/chat/completions';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"messages":[{"role":"user","content":"Can you explain the theory of relativity in simple terms?"}]}'
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

	url := "https://openrouter.ai/api/v1/chat/completions"

	payload := strings.NewReader("{\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": \"Can you explain the theory of relativity in simple terms?\"\n    }\n  ]\n}")

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

url = URI("https://openrouter.ai/api/v1/chat/completions")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": \"Can you explain the theory of relativity in simple terms?\"\n    }\n  ]\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/chat/completions")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": \"Can you explain the theory of relativity in simple terms?\"\n    }\n  ]\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/chat/completions', [
  'body' => '{
  "messages": [
    {
      "role": "user",
      "content": "Can you explain the theory of relativity in simple terms?"
    }
  ]
}',
  'headers' => [
    'Authorization' => 'Bearer <token>',
    'Content-Type' => 'application/json',
  ],
]);

echo $response->getBody();
```

### C\#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/chat/completions");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": \"Can you explain the theory of relativity in simple terms?\"\n    }\n  ]\n}", ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = [
  "Authorization": "Bearer <token>",
  "Content-Type": "application/json"
]
let parameters = ["messages": [
    [
      "role": "user",
      "content": "Can you explain the theory of relativity in simple terms?"
    ]
  ]] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/chat/completions")! as URL,
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

---

## Request Body Parameters

### Root Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `messages` | Array[Message] | Yes | -- | Chat conversation messages |
| `model` | string | No | -- | Model identifier to use for completion |
| `models` | Array[string] | No | -- | Alternative model list for fallback routing |
| `provider` | Object | No | null | Routing preferences when multiple providers available |
| `plugins` | Array[Object] | No | -- | Plugins to enable with settings |
| `route` | string (enum) | No | null | Route strategy: `fallback`, `sort` |
| `user` | string | No | -- | User identifier for tracking |
| `session_id` | string | No | -- | Unique identifier for related requests (max 128 chars) |
| `trace` | Object | No | -- | Observability and tracing metadata |
| `frequency_penalty` | number | No | null | Penalty for repeated tokens (-2.0 to 2.0) |
| `presence_penalty` | number | No | null | Penalty for token presence (-2.0 to 2.0) |
| `temperature` | number | No | 1 | Sampling temperature (0.0 to 2.0) |
| `top_p` | number | No | 1 | Nucleus sampling parameter (0.0 to 1.0) |
| `top_logprobs` | number | No | null | Return top log probabilities |
| `logprobs` | boolean | No | null | Include log probability information |
| `logit_bias` | Object | No | null | Adjust logits for specific tokens |
| `max_tokens` | number | No | null | Maximum completion tokens |
| `max_completion_tokens` | number | No | null | Alternative max tokens parameter |
| `stop` | string \| Array[string] | No | null | Stop sequences for generation |
| `seed` | integer | No | null | Random seed for reproducibility |
| `stream` | boolean | No | false | Enable streaming responses |
| `stream_options` | Object | No | null | Streaming configuration options |
| `response_format` | Object | No | -- | Output format specification |
| `tool_choice` | string \| Object | No | -- | Tool selection strategy |
| `tools` | Array[Object] | No | -- | Available tool definitions |
| `reasoning` | Object | No | -- | Reasoning configuration |
| `metadata` | Object | No | -- | Custom metadata key-value pairs |
| `debug` | Object | No | -- | Debug options |
| `image_config` | Object | No | -- | Image generation configuration |
| `modalities` | Array[string] | No | -- | Output modalities: `text`, `image` |

### Message Object Structure

Messages can be one of: **SystemMessage**, **UserMessage**, **DeveloperMessage**, **AssistantMessage**, **ToolResponseMessage**

#### SystemMessage

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | Yes | Literal: `"system"` |
| `content` | string \| Array[TextItem] | Yes | System instructions |
| `name` | string | No | Message identifier |

#### UserMessage

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | Yes | Literal: `"user"` |
| `content` | string \| Array[ContentItem] | Yes | User input (text, images, audio, video) |
| `name` | string | No | Message identifier |

#### DeveloperMessage

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | Yes | Literal: `"developer"` |
| `content` | string \| Array[TextItem] | Yes | Developer-level instructions |
| `name` | string | No | Message identifier |

#### AssistantMessage

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | Yes | Literal: `"assistant"` |
| `content` | string \| Array[ContentItem] \| null | No | Assistant response content |
| `name` | string | No | Message identifier |
| `tool_calls` | Array[ToolCall] | No | Function calls made by assistant |
| `refusal` | string \| null | No | Refusal explanation |
| `reasoning` | string \| null | No | Reasoning content |
| `reasoning_details` | Array[ReasoningDetail] | No | Detailed reasoning breakdown |
| `images` | Array[ImageItem] | No | Generated images |

#### ToolResponseMessage

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | Yes | Literal: `"tool"` |
| `content` | string \| Array[ContentItem] | Yes | Tool execution result |
| `tool_call_id` | string | Yes | ID of tool call being responded to |

### Content Items

#### Text Content Item

```yaml
type: "text" (required)
text: string (required)
cache_control:
  type: "ephemeral"
  ttl: "5m" | "1h"
```

#### Image URL Content Item

```yaml
type: "image_url" (required)
image_url:
  url: string (required)
  detail: "auto" | "low" | "high"
```

#### Input Audio Content Item

```yaml
type: "input_audio" (required)
input_audio:
  data: string (required, base64)
  format: string (required)
```

#### Input Video Content Item

```yaml
type: "input_video" (required)
video_url:
  url: string (required)
```

#### Video URL Content Item

```yaml
type: "video_url" (required)
video_url:
  url: string (required)
```

### Provider Configuration

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `allow_fallbacks` | boolean | true | Use backup providers if primary unavailable |
| `require_parameters` | boolean | false | Filter to providers supporting all parameters |
| `data_collection` | string | "allow" | Data collection permission: `allow` or `deny` |
| `order` | Array[string] | null | Ordered provider slug list for routing |
| `only` | Array[string] | null | Whitelist of provider slugs |
| `ignore` | Array[string] | null | Blacklist of provider slugs |
| `quantizations` | Array[string] | null | Quantization filters: `int4`, `int8`, `fp4`, `fp6`, `fp8`, `fp16`, `bf16`, `fp32`, `unknown` |
| `sort` | string \| Object | null | Sorting strategy: `price`, `throughput`, `latency` |
| `max_price` | Object | null | Maximum price per million tokens |
| `preferred_min_throughput` | number \| Object | null | Minimum tokens/second threshold |
| `preferred_max_latency` | number \| Object | null | Maximum latency in seconds |
| `zdr` | boolean | null | Zero data retention requirement |
| `enforce_distillable_text` | boolean | null | Text distillation enforcement |

### Response Format Configuration

**Text Format:**
```yaml
type: "text"
```

**JSON Object Format:**
```yaml
type: "json_object"
```

**JSON Schema Format:**
```yaml
type: "json_schema"
json_schema:
  name: string (required)
  description: string (optional)
  schema: object (required, JSON Schema)
  strict: boolean (optional)
```

**Grammar Format:**
```yaml
type: "grammar"
grammar: string (required)
```

**Python Format:**
```yaml
type: "python"
```

### Tools Configuration

```yaml
type: "function" (required)
function:
  name: string (required)
  description: string (optional)
  parameters: object (required, JSON Schema)
  strict: boolean (optional)
```

**Tool Choice Options:** `"none"`, `"auto"`, `"required"`, or named function object:
```json
{
  "type": "function",
  "function": {
    "name": "my_function"
  }
}
```

### Reasoning Configuration

| Field | Type | Enum Values | Description |
|-------|------|-------------|-------------|
| `effort` | string | `xhigh`, `high`, `medium`, `low`, `minimal`, `none` | Reasoning intensity level |
| `summary` | string | `auto`, `concise`, `detailed` | Summary verbosity |

### Plugins Configuration

Available plugins:

- **`auto-router`**: Auto-routing with `allowed_models`
- **`moderation`**: Content moderation
- **`web`**: Web search (engines: `native`, `exa`)
- **`file-parser`**: File parsing (PDF engines: `mistral-ocr`, `pdf-text`, `native`)
- **`response-healing`**: Response validation and repair

### Stream Options

| Field | Type | Description |
|-------|------|-------------|
| `include_usage` | boolean | Include token usage in stream |

### Trace Metadata

| Field | Type | Description |
|-------|------|-------------|
| `trace_id` | string | Unique trace identifier |
| `trace_name` | string | Trace name for grouping |
| `span_name` | string | Span identifier within trace |
| `generation_name` | string | Generation identifier |
| `parent_span_id` | string | Parent span reference |

---

## Response Schema

### Success Response (200)

```yaml
id: string
object: "chat.completion"
created: number (Unix timestamp)
model: string
choices:
  - finish_reason: "stop" | "tool_calls" | "length" | "content_filter" | "error" | null
    index: number
    message:
      role: "assistant"
      content: string | Array[ContentItem] | null
      tool_calls:
        - id: string
          type: "function"
          function:
            name: string
            arguments: string (JSON)
      refusal: string | null
      reasoning: string | null
      reasoning_details:
        - type: "reasoning.summary" | "reasoning.encrypted" | "reasoning.text"
          summary: string (for summary type)
          data: string (for encrypted type)
          text: string | null (for text type)
          signature: string | null (for text type)
          id: string | null
          format: string | null
          index: number
      images:
        - image_url:
            url: string
    logprobs:
      content:
        - token: string
          logprob: number
          bytes: Array[number] | null
          top_logprobs:
            - token: string
              logprob: number
              bytes: Array[number] | null
      refusal: Array[LogprobItem] | null
system_fingerprint: string | null
usage:
  prompt_tokens: number
  completion_tokens: number
  total_tokens: number
  prompt_tokens_details:
    cached_tokens: number
    cache_write_tokens: number
    audio_tokens: number
    video_tokens: number
  completion_tokens_details:
    reasoning_tokens: number | null
    audio_tokens: number | null
    accepted_prediction_tokens: number | null
    rejected_prediction_tokens: number | null
```

### Error Responses

| Status | Description |
|--------|-------------|
| **400** | Bad request - invalid parameters |
| **401** | Unauthorized - invalid API key |
| **429** | Too many requests - rate limit exceeded |
| **500** | Internal server error |
