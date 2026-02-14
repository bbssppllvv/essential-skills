# Preview the Impact of ZDR on Available Endpoints

`GET https://openrouter.ai/api/v1/endpoints/zdr`

Preview how Zero Data Retention (ZDR) impacts the available endpoints through OpenRouter's infrastructure.

## Request

### Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | string | Yes | API key as bearer token in Authorization header |

## Response

### Status Codes

| Code | Description |
|------|-------------|
| 200 | Returns a list of ZDR-compatible endpoints |
| 500 | Internal Server Error - Unexpected server error |

### Response Schema (200)

Returns an array of endpoint objects:

```json
{
  "data": [
    {
      "name": "string",
      "model_id": "string",
      "model_name": "string",
      "context_length": 0,
      "pricing": { },
      "provider_name": "string",
      "tag": "string",
      "quantization": { },
      "max_completion_tokens": 0,
      "max_prompt_tokens": 0,
      "supported_parameters": [],
      "status": 0,
      "uptime_last_30m": 0,
      "supports_implicit_caching": false,
      "latency_last_30m": { },
      "throughput_last_30m": { }
    }
  ]
}
```

### Endpoint Object Fields

| Field | Type | Description |
|-------|------|-------------|
| name | string | Endpoint identifier |
| model_id | string | Unique model identifier (permaslug) |
| model_name | string | Human-readable model name |
| context_length | number | Maximum context window size |
| pricing | object | Multi-tier pricing structure including prompt, completion, request, image, audio, web_search, reasoning, and cache operations |
| provider_name | enum | Provider from 60+ supported vendors |
| tag | string | Classification tag |
| quantization | object | Model quantization details |
| max_completion_tokens | number \| null | Maximum output tokens |
| max_prompt_tokens | number \| null | Maximum input tokens |
| supported_parameters | array | List of supported API parameters |
| status | enum | Endpoint health status (0, -1, -2, -3, -5, -10) |
| uptime_last_30m | number \| null | Availability percentage over 30 minutes |
| supports_implicit_caching | boolean | Caching capability |
| latency_last_30m | object | Latency percentile statistics (p50, p75, p90, p99) |
| throughput_last_30m | object | Throughput percentile statistics (p50, p75, p90, p99) |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/endpoints/zdr"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/endpoints/zdr';
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

	url := "https://openrouter.ai/api/v1/endpoints/zdr"

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

url = URI("https://openrouter.ai/api/v1/endpoints/zdr")

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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/endpoints/zdr")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/endpoints/zdr', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/endpoints/zdr");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/endpoints/zdr")! as URL,
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
