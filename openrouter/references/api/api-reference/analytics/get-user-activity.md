# Get User Activity Grouped by Endpoint

Returns user activity data grouped by endpoint for the last 30 (completed) UTC days.

**Endpoint:** `GET https://openrouter.ai/api/v1/activity`

**Authentication:** Requires a management API key passed as a Bearer token in the Authorization header.

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date` | string | No | Filter by a single UTC date in the last 30 days (YYYY-MM-DD format). |

## Headers

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `Authorization` | string | Yes | API key as bearer token in Authorization header |

---

## Response

### 200 - Success

Returns user activity data grouped by endpoint.

**Response Body:**

```json
{
  "data": [
    {
      "date": "2025-01-15",
      "model": "openai/gpt-4.1",
      "model_permaslug": "openai/gpt-4.1-2025-04-14",
      "endpoint_id": "string",
      "provider_name": "string",
      "usage": 0,
      "byok_usage_inference": 0,
      "requests": 0,
      "prompt_tokens": 0,
      "completion_tokens": 0,
      "reasoning_tokens": 0
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data` | array of ActivityItem | List of activity items |
| `data[].date` | string | Date of the activity (YYYY-MM-DD format) |
| `data[].model` | string | Model slug (e.g., "openai/gpt-4.1") |
| `data[].model_permaslug` | string | Model permaslug (e.g., "openai/gpt-4.1-2025-04-14") |
| `data[].endpoint_id` | string | Unique identifier for the endpoint |
| `data[].provider_name` | string | Name of the provider serving this endpoint |
| `data[].usage` | number | Total cost in USD (OpenRouter credits spent) |
| `data[].byok_usage_inference` | number | BYOK inference cost in USD (external credits spent) |
| `data[].requests` | number | Number of requests made |
| `data[].prompt_tokens` | number | Total prompt tokens used |
| `data[].completion_tokens` | number | Total completion tokens generated |
| `data[].reasoning_tokens` | number | Total reasoning tokens used |

### Error Responses

| Status Code | Description |
|-------------|-------------|
| 400 | Bad Request - Invalid date format or date range |
| 401 | Unauthorized - Authentication required or invalid credentials |
| 403 | Forbidden - Only management keys can fetch activity |
| 500 | Internal Server Error - Unexpected server error |

---

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/activity"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/activity';
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

	url := "https://openrouter.ai/api/v1/activity"

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

url = URI("https://openrouter.ai/api/v1/activity")

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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/activity")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/activity', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/activity");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/activity")! as URL,
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
