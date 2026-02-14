# Get Current API Key

Get information on the API key associated with the current authentication session.

## Endpoint

**Method:** GET
**URL:** `https://openrouter.ai/api/v1/key`

## Authentication

Bearer token required in Authorization header.

## Request Parameters

| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|
| Authorization | header | string | Yes | API key as bearer token |

## Response (200 OK)

**Root Object:**

- `data` (object, required): Current API key information

**Data Object Properties:**

| Field | Type | Description |
|-------|------|-------------|
| label | string | Human-readable label for the API key |
| limit | number\|null | Spending limit in USD |
| usage | number | Total OpenRouter credit usage (USD) |
| usage_daily | number | Usage for current UTC day (USD) |
| usage_weekly | number | Usage for current UTC week (USD) |
| usage_monthly | number | Usage for current UTC month (USD) |
| byok_usage | number | Total external BYOK usage (USD) |
| byok_usage_daily | number | BYOK usage for current UTC day (USD) |
| byok_usage_weekly | number | BYOK usage for current UTC week (USD) |
| byok_usage_monthly | number | BYOK usage for current UTC month (USD) |
| is_free_tier | boolean | Whether this is a free tier key |
| is_management_key | boolean | Whether this is a management key |
| is_provisioning_key | boolean | Whether this is a provisioning key |
| limit_remaining | number\|null | Remaining spending limit (USD) |
| limit_reset | string\|null | Type of limit reset |
| include_byok_in_limit | boolean | Include BYOK in credit limit |
| expires_at | string\|null | ISO 8601 expiration timestamp |
| rate_limit | object | Legacy rate limit info (returns -1) |

**Rate Limit Object:**

| Field | Type | Description |
|-------|------|-------------|
| requests | number | Requests allowed per interval |
| interval | string | Rate limit interval |
| note | string | Note about the rate limit |

## Error Responses

- **401:** Unauthorized - Authentication required or invalid credentials
- **500:** Internal Server Error - Unexpected server error

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/key"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/key';
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

	url := "https://openrouter.ai/api/v1/key"

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

url = URI("https://openrouter.ai/api/v1/key")

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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/key")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/key', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/key");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/key")! as URL,
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
