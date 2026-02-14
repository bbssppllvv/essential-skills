# List All Embeddings Models

`GET https://openrouter.ai/api/v1/embeddings/models`

Returns a list of all available embeddings models and their properties.

## Request

### Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | string | Yes | API key as bearer token in Authorization header |

## Response

### Status Codes

| Code | Description |
|------|-------------|
| 200 | Successful response with models list |
| 400 | Invalid request parameters |
| 500 | Server error |

### Response Schema (200)

Returns a `ModelsListResponse` containing an array of `Model` objects:

```json
{
  "data": [
    {
      "id": "string",
      "canonical_slug": "string",
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
| id | string | Yes | Unique model identifier |
| canonical_slug | string | Yes | Canonical slug reference |
| name | string | Yes | Display name |
| created | number | Yes | Unix timestamp |
| description | string | Yes | Model details |
| pricing | object | Yes | Cost structure with prompt, completion, requests, images, audio, caching, web search, reasoning rates |
| context_length | number \| null | Yes | Maximum context length in tokens |
| architecture | object | Yes | Tokenizer, instruction format, modalities |
| top_provider | object | Yes | Context length and moderation info |
| per_request_limits | object | Yes | Max prompt and completion tokens |
| supported_parameters | array | Yes | Available configuration options |
| default_parameters | object | Yes | Default temperature, top_p, frequency_penalty values |
| expiration_date | string \| null | No | Removal timeline (if applicable) |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/embeddings/models"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/embeddings/models';
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

	url := "https://openrouter.ai/api/v1/embeddings/models"

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

url = URI("https://openrouter.ai/api/v1/embeddings/models")

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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/embeddings/models")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/embeddings/models', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/embeddings/models");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/embeddings/models")! as URL,
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
