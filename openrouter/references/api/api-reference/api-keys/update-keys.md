# Update API Key

Update an existing API key. Management key required.

## Endpoint

**Method:** PATCH
**URL:** `https://openrouter.ai/api/v1/keys/{hash}`

## Authentication

Management key required as Bearer token in Authorization header.

## Request Parameters

### Path Parameter

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| hash | string | Yes | The hash identifier of the API key to update |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | No | New name for the API key |
| disabled | boolean | No | Whether to disable the API key |
| limit | number\|null | No | New spending limit for the API key in USD |
| limit_reset | string\|null | No | New limit reset type for the API key (daily, weekly, monthly, or null for no reset) |
| include_byok_in_limit | boolean | No | Whether to include BYOK usage in the limit |

## Response (200 OK)

Returns an object with a `data` field containing:

| Field | Type | Description |
|-------|------|-------------|
| hash | string | Unique hash identifier for the API key |
| name | string | Name of the API key |
| label | string | Human-readable label for the API key |
| disabled | boolean | Whether the API key is disabled |
| limit | number\|null | Spending limit for the API key in USD |
| limit_remaining | number\|null | Remaining spending limit in USD |
| limit_reset | string\|null | Type of limit reset for the API key |
| include_byok_in_limit | boolean | Whether to include external BYOK usage in the credit limit |
| usage | number | Total OpenRouter credit usage (in USD) |
| usage_daily | number | OpenRouter credit usage (in USD) for the current UTC day |
| usage_weekly | number | OpenRouter credit usage (in USD) for the current UTC week |
| usage_monthly | number | OpenRouter credit usage (in USD) for the current UTC month |
| byok_usage | number | Total external BYOK usage (in USD) |
| byok_usage_daily | number | External BYOK usage (in USD) for the current UTC day |
| byok_usage_weekly | number | External BYOK usage (in USD) for the current UTC week |
| byok_usage_monthly | number | External BYOK usage (in USD) for the current UTC month |
| created_at | string | ISO 8601 timestamp of when the API key was created |
| updated_at | string\|null | ISO 8601 timestamp of when the API key was last updated |
| expires_at | string\|null | ISO 8601 UTC timestamp when the API key expires |

## Error Responses

- **400:** Bad Request - Invalid request parameters
- **401:** Unauthorized - Missing or invalid authentication
- **404:** Not Found - API key does not exist
- **429:** Too Many Requests - Rate limit exceeded
- **500:** Internal Server Error

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943"

payload = {
    "name": "Updated API Key Name",
    "disabled": False,
    "limit": 75,
    "limit_reset": "daily",
    "include_byok_in_limit": True
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
const url = 'https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943';
const options = {
  method: 'PATCH',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"name":"Updated API Key Name","disabled":false,"limit":75,"limit_reset":"daily","include_byok_in_limit":true}'
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

	url := "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943"

	payload := strings.NewReader("{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}")

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

url = URI("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Patch.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.patch("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('PATCH', 'https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943', [
  'body' => '{
  "name": "Updated API Key Name",
  "disabled": false,
  "limit": 75,
  "limit_reset": "daily",
  "include_byok_in_limit": true
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

var client = new RestClient("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943");
var request = new RestRequest(Method.PATCH);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}", ParameterType.RequestBody);
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
  "name": "Updated API Key Name",
  "disabled": false,
  "limit": 75,
  "limit_reset": "daily",
  "include_byok_in_limit": true
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")! as URL,
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
