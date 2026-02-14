# Create a Guardrail

`POST https://openrouter.ai/api/v1/guardrails`

Create a new guardrail for the authenticated user. Requires a management API key.

## Authentication

Requires a Management API key passed as a bearer token in the `Authorization` header.

## Headers

| Header | Value |
|--------|-------|
| Authorization | Bearer `<token>` |
| Content-Type | application/json |

## Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Name for the new guardrail |
| description | string or null | No | Description of the guardrail |
| limit_usd | number or null | No | Spending limit in USD |
| reset_interval | string or null | No | Interval at which the limit resets (`daily`, `weekly`, `monthly`) |
| allowed_providers | array or null | No | List of allowed provider IDs |
| allowed_models | array or null | No | Array of model identifiers (slug or canonical_slug accepted) |
| enforce_zdr | boolean or null | No | Whether to enforce zero data retention |

## Response (201 Created)

| Field | Type | Description |
|-------|------|-------------|
| id | string (uuid) | Unique identifier for the guardrail |
| name | string | Name of the guardrail |
| description | string or null | Description of the guardrail |
| limit_usd | number or null | Spending limit in USD |
| reset_interval | string or null | Interval at which the limit resets (`daily`, `weekly`, `monthly`) |
| allowed_providers | array or null | List of allowed provider IDs |
| allowed_models | array or null | Array of model canonical_slugs (immutable identifiers) |
| enforce_zdr | boolean or null | Whether to enforce zero data retention |
| created_at | string | ISO 8601 timestamp of when the guardrail was created |
| updated_at | string or null | ISO 8601 timestamp of when the guardrail was last updated |

## Error Responses

| Code | Description |
|------|-------------|
| 400 | Bad Request - Invalid request parameters |
| 401 | Unauthorized - Missing or invalid authentication |
| 500 | Internal Server Error |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/guardrails"

payload = {
    "name": "My New Guardrail",
    "description": "A guardrail for limiting API usage",
    "limit_usd": 50,
    "reset_interval": "monthly",
    "allowed_providers": ["openai", "anthropic", "deepseek"],
    "allowed_models": None,
    "enforce_zdr": False
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
const url = 'https://openrouter.ai/api/v1/guardrails';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"name":"My New Guardrail","description":"A guardrail for limiting API usage","limit_usd":50,"reset_interval":"monthly","allowed_providers":["openai","anthropic","deepseek"],"allowed_models":null,"enforce_zdr":false}'
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

	url := "https://openrouter.ai/api/v1/guardrails"

	payload := strings.NewReader("{\n  \"name\": \"My New Guardrail\",\n  \"description\": \"A guardrail for limiting API usage\",\n  \"limit_usd\": 50,\n  \"reset_interval\": \"monthly\",\n  \"allowed_providers\": [\n    \"openai\",\n    \"anthropic\",\n    \"deepseek\"\n  ],\n  \"allowed_models\": null,\n  \"enforce_zdr\": false\n}")

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

url = URI("https://openrouter.ai/api/v1/guardrails")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"name\": \"My New Guardrail\",\n  \"description\": \"A guardrail for limiting API usage\",\n  \"limit_usd\": 50,\n  \"reset_interval\": \"monthly\",\n  \"allowed_providers\": [\n    \"openai\",\n    \"anthropic\",\n    \"deepseek\"\n  ],\n  \"allowed_models\": null,\n  \"enforce_zdr\": false\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/guardrails")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"name\": \"My New Guardrail\",\n  \"description\": \"A guardrail for limiting API usage\",\n  \"limit_usd\": 50,\n  \"reset_interval\": \"monthly\",\n  \"allowed_providers\": [\n    \"openai\",\n    \"anthropic\",\n    \"deepseek\"\n  ],\n  \"allowed_models\": null,\n  \"enforce_zdr\": false\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/guardrails', [
  'body' => '{
  "name": "My New Guardrail",
  "description": "A guardrail for limiting API usage",
  "limit_usd": 50,
  "reset_interval": "monthly",
  "allowed_providers": [
    "openai",
    "anthropic",
    "deepseek"
  ],
  "allowed_models": null,
  "enforce_zdr": false
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

var client = new RestClient("https://openrouter.ai/api/v1/guardrails");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"name\": \"My New Guardrail\",\n  \"description\": \"A guardrail for limiting API usage\",\n  \"limit_usd\": 50,\n  \"reset_interval\": \"monthly\",\n  \"allowed_providers\": [\n    \"openai\",\n    \"anthropic\",\n    \"deepseek\"\n  ],\n  \"allowed_models\": null,\n  \"enforce_zdr\": false\n}", ParameterType.RequestBody);
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
  "name": "My New Guardrail",
  "description": "A guardrail for limiting API usage",
  "limit_usd": 50,
  "reset_interval": "monthly",
  "allowed_providers": ["openai", "anthropic", "deepseek"],
  "allowed_models": ,
  "enforce_zdr": false
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/guardrails")! as URL,
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
