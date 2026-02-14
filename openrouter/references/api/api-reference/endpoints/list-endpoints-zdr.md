# Preview the Impact of ZDR on Available Endpoints

## Endpoint Overview

This API provides a GET request to preview how Zero Downtime Replacement (ZDR) affects endpoint availability.

**URL:** `https://openrouter.ai/api/v1/endpoints/zdr`

**Method:** GET

## Authentication

The endpoint requires bearer token authentication via the Authorization header.

## Request Parameters

- **Authorization** (header, required): API key formatted as a bearer token

## Response

### Success Response (200)

Returns a JSON object containing an array of endpoint objects with the following structure:

| Field | Type | Description |
|-------|------|-------------|
| name | string | Endpoint identifier |
| model_id | string | Unique model identifier (permaslug) |
| model_name | string | Human-readable model name |
| context_length | number | Maximum context window size |
| pricing | object | Pricing details for various operations |
| provider_name | string | Service provider name |
| status | string | Current endpoint status |
| uptime_last_30m | number | Uptime percentage over 30 minutes |
| latency_last_30m | object | Latency percentiles (p50, p75, p90, p99) |
| throughput_last_30m | object | Throughput metrics over 30 minutes |

### Error Response (500)

Returns an internal server error for unexpected issues.

## Code Examples

### Python
```python
import requests

url = "https://openrouter.ai/api/v1/endpoints/zdr"
headers = {"Authorization": "Bearer <token>"}
response = requests.get(url, headers=headers)
print(response.json())
```

### JavaScript
```javascript
const url = 'https://openrouter.ai/api/v1/endpoints/zdr';
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
	url := "https://openrouter.ai/api/v1/endpoints/zdr"
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

url = URI("https://openrouter.ai/api/v1/endpoints/zdr")
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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/endpoints/zdr")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP
```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();
$response = $client->request('GET', 'https://openrouter.ai/api/v1/endpoints/zdr', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);
echo $response->getBody();
```

### C#
```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/endpoints/zdr");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift
```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]
let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/endpoints/zdr")! as URL,
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
