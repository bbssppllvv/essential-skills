# List All Endpoints for a Model

`GET https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints`

Retrieves all available endpoints for a specific model, identified by the author and slug parameters.

## Request

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| author | string | Yes | Model author identifier |
| slug | string | Yes | Model slug identifier |

### Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | string | Yes | API key as bearer token in Authorization header |

## Response

### Status Codes

| Code | Description |
|------|-------------|
| 200 | Returns a list of endpoints |
| 404 | Not Found - Model does not exist |
| 500 | Internal Server Error - Unexpected server error |

### Response Schema (200)

The response contains model metadata and available endpoints:

```json
{
  "id": "string",
  "name": "string",
  "created": 0,
  "description": "string",
  "architecture": {
    "tokenizer": "string",
    "instruct_type": "string",
    "modality": "string",
    "input_modalities": [],
    "output_modalities": []
  },
  "endpoints": []
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique model identifier |
| name | string | Display name |
| created | number | Unix timestamp |
| description | string | Model description |
| architecture | object | Technical specifications including tokenizer, instruct_type, modality, input_modalities, output_modalities |
| endpoints | PublicEndpoint[] | Array of available endpoints (see below) |

### PublicEndpoint Object

| Field | Type | Description |
|-------|------|-------------|
| name | string | Endpoint name |
| model_id | string | Unique model identifier (permaslug) |
| model_name | string | Human-readable model name |
| context_length | number | Maximum context window size |
| pricing | object | Pricing details for various operations |
| provider_name | string | Service provider name |
| tag | string | Classification tag |
| quantization | object | Model quantization details |
| max_completion_tokens | number \| null | Maximum output tokens |
| max_prompt_tokens | number \| null | Maximum input tokens |
| supported_parameters | array | List of supported API parameters |
| status | string | Current endpoint status |
| uptime_last_30m | number \| null | Uptime percentage over 30 minutes |
| supports_implicit_caching | boolean | Caching capability |
| latency_last_30m | object | Latency percentiles (p50, p75, p90, p99) |
| throughput_last_30m | object | Throughput metrics over 30 minutes (p50, p75, p90, p99) |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/models/author/slug/endpoints"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/models/author/slug/endpoints';
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

	url := "https://openrouter.ai/api/v1/models/author/slug/endpoints"

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

url = URI("https://openrouter.ai/api/v1/models/author/slug/endpoints")

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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/models/author/slug/endpoints")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/models/author/slug/endpoints', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/models/author/slug/endpoints");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/models/author/slug/endpoints")! as URL,
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
