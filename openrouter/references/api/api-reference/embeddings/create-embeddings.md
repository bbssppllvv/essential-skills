# Submit an Embedding Request

`POST https://openrouter.ai/api/v1/embeddings`

Submits an embedding request to the embeddings router.

## Request

### Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | string | Yes | API key as bearer token in Authorization header |
| Content-Type | string | Yes | `application/json` |

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| input | string \| string[] \| number[] \| number[][] \| object[] | Yes | Text or content to embed |
| model | string | Yes | Model identifier |
| encoding_format | string | No | Output format (`float` or `base64`) |
| dimensions | integer | No | Desired embedding dimensions |
| user | string | No | User identifier |
| provider | object | No | Provider preferences and routing |
| input_type | string | No | Input type specification |

### Input Format Options

- **Single string:** `"The quick brown fox jumps over the lazy dog"`
- **Array of strings:** `["text1", "text2"]`
- **Array of token arrays:** `[[1, 2, 3], [4, 5, 6]]`
- **Complex content:** Objects with `content` array containing text/image items

## Response

### Status Codes

| Code | Description |
|------|-------------|
| 200 | Embedding response successful |
| 400 | Invalid request parameters |
| 401 | Authentication required or invalid |
| 402 | Insufficient credits/quota |
| 404 | Resource not found |
| 429 | Rate limit exceeded |
| 500 | Server error |
| 502 | Provider API failure |
| 503 | Service unavailable |

### Response Schema (200)

| Field | Type | Description |
|-------|------|-------------|
| id | string | Request identifier |
| object | string | Always `"list"` |
| data | array | Array of embedding objects |
| data[].object | string | Always `"embedding"` |
| data[].embedding | number[] \| string | The embedding vector |
| data[].index | number | Position in input array |
| model | string | Model used |
| usage.prompt_tokens | number | Tokens processed |
| usage.total_tokens | number | Total tokens |
| usage.cost | number | Cost in USD |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/embeddings"

payload = {
    "input": "The quick brown fox jumps over the lazy dog",
    "model": "text-embedding-ada-002"
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
const url = 'https://openrouter.ai/api/v1/embeddings';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"input":"The quick brown fox jumps over the lazy dog","model":"text-embedding-ada-002"}'
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

	url := "https://openrouter.ai/api/v1/embeddings"

	payload := strings.NewReader("{\n  \"input\": \"The quick brown fox jumps over the lazy dog\",\n  \"model\": \"text-embedding-ada-002\"\n}")

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

url = URI("https://openrouter.ai/api/v1/embeddings")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"input\": \"The quick brown fox jumps over the lazy dog\",\n  \"model\": \"text-embedding-ada-002\"\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/embeddings")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"input\": \"The quick brown fox jumps over the lazy dog\",\n  \"model\": \"text-embedding-ada-002\"\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/embeddings', [
  'body' => '{
  "input": "The quick brown fox jumps over the lazy dog",
  "model": "text-embedding-ada-002"
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

var client = new RestClient("https://openrouter.ai/api/v1/embeddings");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"input\": \"The quick brown fox jumps over the lazy dog\",\n  \"model\": \"text-embedding-ada-002\"\n}", ParameterType.RequestBody);
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
  "input": "The quick brown fox jumps over the lazy dog",
  "model": "text-embedding-ada-002"
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/embeddings")! as URL,
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
