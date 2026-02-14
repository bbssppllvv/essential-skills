# Update a Guardrail

`PATCH https://openrouter.ai/api/v1/guardrails/{id}`

Update an existing guardrail. Requires a management API key.

## Authentication

Requires a Management API key passed as a bearer token in the `Authorization` header.

## Headers

| Header | Value |
|--------|-------|
| Authorization | Bearer `<token>` |
| Content-Type | application/json |

## Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | string (uuid) | Yes | The unique identifier of the guardrail to update |

## Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | No | New name for the guardrail |
| description | string or null | No | New description for the guardrail |
| limit_usd | number or null | No | New spending limit in USD |
| reset_interval | string or null | No | Interval at which the limit resets (`daily`, `weekly`, `monthly`) |
| allowed_providers | array or null | No | New list of allowed provider IDs |
| allowed_models | array or null | No | Array of model identifiers (slug or canonical_slug accepted) |
| enforce_zdr | boolean or null | No | Whether to enforce zero data retention |

## Response (200 OK)

The response contains a `data` object with the updated guardrail:

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
| 404 | Not Found - Guardrail does not exist |
| 500 | Internal Server Error |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000"

payload = {
    "name": "Updated Guardrail Name",
    "description": "Updated description",
    "limit_usd": 75,
    "reset_interval": "weekly"
}
headers = {
    "Authorization": "Bearer <token>",
    "Content-Type": "application/json"
}

response = requests.patch(url, json=payload, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000';
const options = {
  method: 'PATCH',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"name":"Updated Guardrail Name","description":"Updated description","limit_usd":75,"reset_interval":"weekly"}'
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

	url := "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000"

	payload := strings.NewReader("{\n  \"name\": \"Updated Guardrail Name\",\n  \"description\": \"Updated description\",\n  \"limit_usd\": 75,\n  \"reset_interval\": \"weekly\"\n}")

	req, _ := http.NewRequest("PATCH", url, payload)

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

url = URI("https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Patch.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"name\": \"Updated Guardrail Name\",\n  \"description\": \"Updated description\",\n  \"limit_usd\": 75,\n  \"reset_interval\": \"weekly\"\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.patch("https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"name\": \"Updated Guardrail Name\",\n  \"description\": \"Updated description\",\n  \"limit_usd\": 75,\n  \"reset_interval\": \"weekly\"\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('PATCH', 'https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000', [
  'body' => '{
  "name": "Updated Guardrail Name",
  "description": "Updated description",
  "limit_usd": 75,
  "reset_interval": "weekly"
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

var client = new RestClient("https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000");
var request = new RestRequest(Method.PATCH);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"name\": \"Updated Guardrail Name\",\n  \"description\": \"Updated description\",\n  \"limit_usd\": 75,\n  \"reset_interval\": \"weekly\"\n}", ParameterType.RequestBody);
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
  "name": "Updated Guardrail Name",
  "description": "Updated description",
  "limit_usd": 75,
  "reset_interval": "weekly"
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000")! as URL,
                                        cachePolicy: .useProtocolCachePolicy,
                                    timeoutInterval: 10.0)
request.httpMethod = "PATCH"
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
