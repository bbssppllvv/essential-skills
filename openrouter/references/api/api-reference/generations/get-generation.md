# Get Request & Usage Metadata for a Generation

Retrieve request and usage metadata for a specific generation.

**Endpoint:** `GET https://openrouter.ai/api/v1/generation`

**Authentication:** Requires an API key passed as a Bearer token in the Authorization header.

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Generation identifier |

## Headers

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `Authorization` | string | Yes | API key as bearer token in Authorization header |

---

## Response

### 200 - Success

Returns the request metadata for this generation.

**Response Body:**

```json
{
  "data": {
    "id": "string",
    "upstream_id": "string",
    "created_at": "string",
    "model": "string",
    "total_cost": 0,
    "cache_discount": 0,
    "upstream_inference_cost": 0,
    "latency": 0,
    "generation_time": 0,
    "moderation_latency": 0,
    "usage": 0,
    "tokens_prompt": 0,
    "tokens_completion": 0,
    "native_tokens_prompt": 0,
    "native_tokens_completion": 0,
    "native_tokens_completion_images": 0,
    "native_tokens_reasoning": 0,
    "native_tokens_cached": 0,
    "streamed": false,
    "cancelled": false,
    "provider_name": "string",
    "is_byok": false,
    "api_type": "string",
    "router": "string",
    "origin": "string",
    "external_user": "string",
    "app_id": 0,
    "finish_reason": "string",
    "native_finish_reason": "string",
    "num_media_prompt": 0,
    "num_input_audio_prompt": 0,
    "num_media_completion": 0,
    "num_search_results": 0,
    "provider_responses": [
      {
        "id": "string",
        "endpoint_id": "string",
        "model_permaslug": "string",
        "provider_name": "string",
        "status": 0,
        "latency": 0,
        "is_byok": false
      }
    ]
  }
}
```

### Core Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.id` | string | Unique identifier for the generation |
| `data.upstream_id` | string or null | Upstream provider's identifier |
| `data.created_at` | string | ISO 8601 timestamp |
| `data.model` | string | Model identifier |
| `data.total_cost` | number | Total cost of the generation in USD |
| `data.cache_discount` | number or null | Discount applied due to caching |
| `data.upstream_inference_cost` | number or null | Cost charged by upstream provider |

### Usage & Performance Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.latency` | number or null | Total latency in milliseconds |
| `data.generation_time` | number or null | Time taken for generation in milliseconds |
| `data.moderation_latency` | number or null | Moderation latency in milliseconds |
| `data.usage` | number | Usage amount in USD |

### Token Count Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.tokens_prompt` | number or null | Total prompt tokens |
| `data.tokens_completion` | number or null | Total completion tokens |
| `data.native_tokens_prompt` | number or null | Native prompt tokens |
| `data.native_tokens_completion` | number or null | Native completion tokens |
| `data.native_tokens_completion_images` | number or null | Native completion image tokens |
| `data.native_tokens_reasoning` | number or null | Native reasoning tokens |
| `data.native_tokens_cached` | number or null | Native cached tokens |

### Request Detail Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.streamed` | boolean or null | Whether response was streamed |
| `data.cancelled` | boolean or null | Whether generation was cancelled |
| `data.provider_name` | string or null | Provider that served the request |
| `data.is_byok` | boolean | Whether this used bring-your-own-key |
| `data.api_type` | string or null | "completions" or "embeddings" |
| `data.router` | string or null | Router used (e.g., "openrouter/auto") |
| `data.origin` | string | Origin URL |
| `data.external_user` | string or null | External user identifier |
| `data.app_id` | number or null | App identifier |
| `data.finish_reason` | string or null | Why generation finished |
| `data.native_finish_reason` | string or null | Provider-reported finish reason |

### Media & Search Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.num_media_prompt` | number or null | Number of media items in prompt |
| `data.num_input_audio_prompt` | number or null | Number of audio inputs in prompt |
| `data.num_media_completion` | number or null | Number of media items in completion |
| `data.num_search_results` | number or null | Number of search results |

### Provider Responses Array

| Field | Type | Description |
|-------|------|-------------|
| `data.provider_responses[].id` | string | Provider response identifier |
| `data.provider_responses[].endpoint_id` | string | Endpoint identifier |
| `data.provider_responses[].model_permaslug` | string | Model permaslug |
| `data.provider_responses[].provider_name` | string | Provider name |
| `data.provider_responses[].status` | number or null | HTTP status code |
| `data.provider_responses[].latency` | number | Latency in milliseconds |
| `data.provider_responses[].is_byok` | boolean | Whether BYOK was used |

### Error Responses

| Status Code | Description |
|-------------|-------------|
| 401 | Unauthorized - Authentication required or invalid credentials |
| 402 | Payment Required - Insufficient credits or quota to complete request |
| 404 | Not Found - Generation not found |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Unexpected server error |
| 502 | Bad Gateway - Provider/upstream API failure |

---

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/generation"

querystring = {"id":"id"}

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers, params=querystring)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/generation?id=id';
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

	url := "https://openrouter.ai/api/v1/generation?id=id"

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

url = URI("https://openrouter.ai/api/v1/generation?id=id")

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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/generation?id=id")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/generation?id=id', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/generation?id=id");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/generation?id=id")! as URL,
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
