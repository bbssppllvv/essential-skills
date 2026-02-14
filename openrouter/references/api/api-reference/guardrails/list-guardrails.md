# List Guardrails

`GET https://openrouter.ai/api/v1/guardrails`

Retrieves all guardrails associated with the authenticated user. Requires a management API key for authorization.

## Authentication

Requires a Management API key passed as a bearer token in the `Authorization` header.

## Query Parameters

| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|
| offset | query | string | No | Number of records to skip for pagination |
| limit | query | string | No | Maximum number of records to return (max 100) |
| Authorization | header | string | Yes | API key as bearer token in Authorization header |

## Response (200 OK)

The response contains a JSON object with these properties:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| data | array | Yes | List of guardrail objects |
| total_count | number | Yes | Total number of guardrails |

### Guardrail Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string (uuid) | Yes | Unique identifier for the guardrail |
| name | string | Yes | Name of the guardrail |
| description | string or null | No | Description of the guardrail |
| limit_usd | number or null | No | Spending limit in USD |
| reset_interval | string or null | No | Interval at which the limit resets (`daily`, `weekly`, `monthly`) |
| allowed_providers | array or null | No | List of allowed provider IDs |
| allowed_models | array or null | No | Array of model canonical slugs |
| enforce_zdr | boolean or null | No | Whether to enforce zero data retention |
| created_at | string | Yes | ISO 8601 timestamp of when the guardrail was created |
| updated_at | string or null | No | ISO 8601 timestamp of when the guardrail was last updated |

## Error Responses

| Code | Description |
|------|-------------|
| 401 | Unauthorized - Missing or invalid authentication |
| 500 | Internal Server Error |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/guardrails"
headers = {"Authorization": "Bearer <token>"}
response = requests.get(url, headers=headers)
print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/guardrails';
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
	url := "https://openrouter.ai/api/v1/guardrails"
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

url = URI("https://openrouter.ai/api/v1/guardrails")
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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/guardrails")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/guardrails', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/guardrails");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/guardrails")! as URL,
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
