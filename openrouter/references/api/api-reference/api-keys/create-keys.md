# Create API Key

Create a new API key for the authenticated user. Management key required.

## Endpoint

**Method:** POST
**URL:** `https://openrouter.ai/api/v1/keys`

## Authentication

Management key required as Bearer token in Authorization header.

## Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Name for the new API key |
| limit | number\|null | No | Optional spending limit for the API key in USD |
| limit_reset | string\|null | No | Type of limit reset for the API key (daily, weekly, monthly, or null for no reset). Resets happen automatically at midnight UTC, and weeks are Monday through Sunday. |
| include_byok_in_limit | boolean | No | Whether to include BYOK usage in the limit |
| expires_at | string\|null | No | Optional ISO 8601 UTC timestamp when the API key should expire |

## Response (201 Created)

| Field | Type | Description |
|-------|------|-------------|
| key | string | The actual API key string (only shown once) |
| data.hash | string | Unique hash identifier |
| data.name | string | Name of the API key |
| data.label | string | Human-readable label |
| data.disabled | boolean | Whether disabled |
| data.limit | number\|null | Spending limit in USD |
| data.limit_remaining | number\|null | Remaining limit in USD |
| data.limit_reset | string\|null | Reset type |
| data.include_byok_in_limit | boolean | BYOK inclusion status |
| data.usage | number | Total OpenRouter credit usage (USD) |
| data.usage_daily | number | Daily OpenRouter credit usage |
| data.usage_weekly | number | Weekly usage |
| data.usage_monthly | number | Monthly usage |
| data.byok_usage | number | Total external BYOK usage |
| data.byok_usage_daily | number | Daily BYOK usage |
| data.byok_usage_weekly | number | Weekly BYOK usage |
| data.byok_usage_monthly | number | Monthly BYOK usage |
| data.created_at | string | ISO 8601 creation timestamp |
| data.updated_at | string\|null | ISO 8601 update timestamp |
| data.expires_at | string\|null | Expiration timestamp |

## Error Responses

- **400:** Bad Request - Invalid parameters
- **401:** Unauthorized - Missing or invalid authentication
- **429:** Too Many Requests - Rate limit exceeded
- **500:** Internal Server Error

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/keys"

payload = {
    "name": "Analytics Service Key",
    "limit": 150,
    "limit_reset": "monthly",
    "include_byok_in_limit": True,
    "expires_at": "2028-06-30T23:59:59Z"
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
const url = 'https://openrouter.ai/api/v1/keys';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"name":"Analytics Service Key","limit":150,"limit_reset":"monthly","include_byok_in_limit":true,"expires_at":"2028-06-30T23:59:59Z"}'
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

	url := "https://openrouter.ai/api/v1/keys"

	payload := strings.NewReader("{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}")

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

url = URI("https://openrouter.ai/api/v1/keys")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/keys")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/keys', [
  'body' => '{
  "name": "Analytics Service Key",
  "limit": 150,
  "limit_reset": "monthly",
  "include_byok_in_limit": true,
  "expires_at": "2028-06-30T23:59:59Z"
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

var client = new RestClient("https://openrouter.ai/api/v1/keys");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}", ParameterType.RequestBody);
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
  "name": "Analytics Service Key",
  "limit": 150,
  "limit_reset": "monthly",
  "include_byok_in_limit": true,
  "expires_at": "2028-06-30T23:59:59Z"
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/keys")! as URL,
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
