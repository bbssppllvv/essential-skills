# List All Models and Their Properties

`GET https://openrouter.ai/api/v1/models`

List all models and their properties via the OpenRouter API.

## Request

### Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | string | Yes | API key as bearer token in Authorization header |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| category | string | No | Filter by use case (programming, roleplay, marketing, marketing/seo, technology, science, translation, legal, finance, health, trivia, academia) |
| supported_parameters | string | No | Filter by supported parameters |
| use_rss | string | No | Enable RSS feed format |
| use_rss_chat_links | string | No | Enable RSS chat links |

## Response

### Status Codes

| Code | Description |
|------|-------------|
| 200 | Returns a list of models or RSS feed |
| 400 | Bad Request - Invalid request parameters |
| 500 | Internal Server Error |

### Response Schema (200)

Returns a `ModelsListResponse` containing an array of `Model` objects:

```json
{
  "data": [
    {
      "id": "string",
      "canonical_slug": "string",
      "hugging_face_id": "string | null",
      "name": "string",
      "created": 0,
      "description": "string",
      "pricing": { },
      "context_length": 0,
      "architecture": { },
      "top_provider": { },
      "per_request_limits": { },
      "supported_parameters": [],
      "default_parameters": { },
      "expiration_date": "string | null"
    }
  ]
}
```

### Model Object Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Yes | Unique identifier for the model |
| canonical_slug | string | Yes | Canonical slug for the model |
| hugging_face_id | string \| null | No | Hugging Face model identifier |
| name | string | Yes | Display name of the model |
| created | number | Yes | Unix timestamp of when the model was created |
| description | string | Yes | Model overview text |
| pricing | PublicPricing | Yes | Pricing structure (see below) |
| context_length | number \| null | Yes | Maximum context length in tokens |
| architecture | ModelArchitecture | Yes | Architecture info (see below) |
| top_provider | TopProviderInfo | Yes | Top provider details (see below) |
| per_request_limits | PerRequestLimits | Yes | Token limits per request (see below) |
| supported_parameters | Parameter[] | Yes | Supported parameter types |
| default_parameters | DefaultParameters | Yes | Default parameter values (see below) |
| expiration_date | string \| null | No | ISO 8601 date string (YYYY-MM-DD) |

### PublicPricing Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| prompt | PublicPricingPrompt | Yes | Prompt token pricing |
| completion | PublicPricingCompletion | Yes | Completion token pricing |
| request | PublicPricingRequest | No | Per-request pricing |
| image | PublicPricingImage | No | Image input pricing |
| image_token | PublicPricingImageToken | No | Image token pricing |
| image_output | PublicPricingImageOutput | No | Image output pricing |
| audio | PublicPricingAudio | No | Audio input pricing |
| audio_output | PublicPricingAudioOutput | No | Audio output pricing |
| input_audio_cache | PublicPricingInputAudioCache | No | Audio cache pricing |
| web_search | PublicPricingWebSearch | No | Web search pricing |
| internal_reasoning | PublicPricingInternalReasoning | No | Internal reasoning pricing |
| input_cache_read | PublicPricingInputCacheRead | No | Cache read pricing |
| input_cache_write | PublicPricingInputCacheWrite | No | Cache write pricing |
| discount | number | No | Discount multiplier |

### ModelArchitecture Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| tokenizer | ModelGroup | Yes | Model framework classification |
| instruct_type | ModelArchitectureInstructType \| null | No | Instruction format type |
| modality | string \| null | Yes | Primary modality of the model |
| input_modalities | InputModality[] | Yes | Supported input modalities |
| output_modalities | OutputModality[] | Yes | Supported output modalities |

### TopProviderInfo Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| context_length | number \| null | Yes | Context length from the top provider |
| max_completion_tokens | number \| null | Yes | Maximum completion tokens |
| is_moderated | boolean | Yes | Whether the top provider moderates content |

### PerRequestLimits Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| prompt_tokens | number | Yes | Maximum prompt tokens per request |
| completion_tokens | number | Yes | Maximum completion tokens per request |

### DefaultParameters Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| temperature | number \| null | No | Default temperature |
| top_p | number \| null | No | Default top_p |
| frequency_penalty | number \| null | No | Default frequency penalty |

### Enum Values

**ModelGroup:** `Router`, `Media`, `Other`, `GPT`, `Claude`, `Gemini`, `Grok`, `Cohere`, `Nova`, `Qwen`, `Yi`, `DeepSeek`, `Mistral`, `Llama2`, `Llama3`, `Llama4`, `PaLM`, `RWKV`, `Qwen3`

**ModelArchitectureInstructType:** `none`, `airoboros`, `alpaca`, `alpaca-modif`, `chatml`, `claude`, `code-llama`, `gemma`, `llama2`, `llama3`, `mistral`, `nemotron`, `neural`, `openchat`, `phi3`, `rwkv`, `vicuna`, `zephyr`, `deepseek-r1`, `deepseek-v3.1`, `qwq`, `qwen3`

**InputModality:** `text`, `image`, `file`, `audio`, `video`

**OutputModality:** `text`, `image`, `embeddings`, `audio`

**Parameter:** `temperature`, `top_p`, `top_k`, `min_p`, `top_a`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `max_tokens`, `logit_bias`, `logprobs`, `top_logprobs`, `seed`, `response_format`, `structured_outputs`, `stop`, `tools`, `tool_choice`, `parallel_tool_calls`, `include_reasoning`, `reasoning`, `reasoning_effort`, `web_search_options`, `verbosity`

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/models"
headers = {"Authorization": "Bearer <token>"}
response = requests.get(url, headers=headers)
print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/models';
const options = {method: 'GET', headers: {Authorization: 'Bearer <token>'}};

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
	"net/http"
	"io"
)

func main() {
	url := "https://openrouter.ai/api/v1/models"
	req, _ := http.NewRequest("GET", url, nil)
	req.Header.Add("Authorization", "Bearer <token>")
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

url = URI("https://openrouter.ai/api/v1/models")
http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true
request = Net::HTTP::Get.new(url)
request["Authorization"] = 'Bearer <token>'
response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/models")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');
$client = new \GuzzleHttp\Client();
$response = $client->request('GET', 'https://openrouter.ai/api/v1/models', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);
echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/models");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]
let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/models")! as URL,
                                        cachePolicy: .useProtocolCachePolicy,
                                    timeoutInterval: 10.0)
request.httpMethod = "GET"
request.allHTTPHeaderFields = headers
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
