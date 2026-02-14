# Get API Key

Get a single API key by hash. Management key required.

## Endpoint

**Method:** GET
**URL:** `https://openrouter.ai/api/v1/keys/{hash}`

## Authentication

Management key required as Bearer token in Authorization header.

## Request Parameters

| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|
| hash | path | string | Yes | The hash identifier of the API key to retrieve |
| Authorization | header | string | Yes | API key as bearer token in Authorization header |

## Response (200 OK)

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
| usage | number | Total OpenRouter credit usage (in USD) for the API key |
| usage_daily | number | OpenRouter credit usage (in USD) for the current UTC day |
| usage_weekly | number | OpenRouter credit usage (in USD) for the current UTC week |
| usage_monthly | number | OpenRouter credit usage (in USD) for the current UTC month |
| byok_usage | number | Total external BYOK usage (in USD) for the API key |
| byok_usage_daily | number | External BYOK usage (in USD) for the current UTC day |
| byok_usage_weekly | number | External BYOK usage (in USD) for the current UTC week |
| byok_usage_monthly | number | External BYOK usage (in USD) for current UTC month |
| created_at | string | ISO 8601 timestamp of when the API key was created |
| updated_at | string\|null | ISO 8601 timestamp of when the API key was last updated |
| expires_at | string\|null | ISO 8601 UTC timestamp when the API key expires, or null if no expiration |

## Error Responses

- **401:** Unauthorized - Missing or invalid authentication
- **404:** Not Found - API key does not exist
- **429:** Too Many Requests - Rate limit exceeded
- **500:** Internal Server Error

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943';
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

	url := "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943"

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

url = URI("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")

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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")! as URL,
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
