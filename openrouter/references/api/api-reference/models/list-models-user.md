# List Models Filtered by User Provider Preferences, Privacy Settings, and Guardrails

## Endpoint Overview

The `GET /models/user` endpoint retrieves a filtered list of AI models based on individual user settings. This includes consideration for "provider preferences, privacy settings, and guardrails."

When accessed via the EU endpoint (`eu.openrouter.ai/api/v1/...`), results are restricted to models complying with EU in-region routing requirements.

## Request Details

**Method:** GET
**URL:** `https://openrouter.ai/api/v1/models/user`

### Required Header
- **Authorization:** Bearer token (API key)

## Response Format

The endpoint returns a JSON object containing:

```json
{
  "data": [
    {
      "id": "string",
      "canonical_slug": "string",
      "name": "string",
      "created": "number",
      "pricing": { /* pricing details */ },
      "context_length": "number",
      "architecture": { /* architecture info */ },
      "top_provider": { /* provider details */ },
      "per_request_limits": { /* token limits */ },
      "supported_parameters": [ /* array of parameters */ ],
      "default_parameters": { /* default settings */ },
      "expiration_date": "string or null"
    }
  ]
}
```

## Status Codes

| Code | Description |
|------|-------------|
| 200 | Successful—returns filtered model list |
| 401 | Unauthorized—missing or invalid authentication |
| 404 | Not Found—no eligible endpoints available |
| 500 | Internal Server Error |

## Code Examples

### Python
```python
import requests

url = "https://openrouter.ai/api/v1/models/user"
headers = {"Authorization": "Bearer <token>"}
response = requests.get(url, headers=headers)
print(response.json())
```

### JavaScript
```javascript
const url = 'https://openrouter.ai/api/v1/models/user';
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
	url := "https://openrouter.ai/api/v1/models/user"
	req, _ := http.NewRequest("GET", url, nil)
	req.Header.Add("Authorization", "Bearer <token>")
	res, _ := http.DefaultClient.Do(req)
	defer res.Body.Close()
	body, _ := io.ReadAll(res.Body)
	fmt.Println(string(body))
}
```

### Ruby
```ruby
require 'uri'
require 'net/http'

url = URI("https://openrouter.ai/api/v1/models/user")
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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/models/user")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP
```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();
$response = $client->request('GET', 'https://openrouter.ai/api/v1/models/user', [
  'headers' => ['Authorization' => 'Bearer <token>'],
]);

echo $response->getBody();
```

### C#
```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/models/user");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift
```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]
let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/models/user")! as URL,
                                  cachePolicy: .useProtocolCachePolicy,
                                  timeoutInterval: 10.0)
request.httpMethod = "GET"
request.allHTTPHeaderFields = headers

let session = URLSession.shared
let dataTask = session.dataTask(with: request as URLRequest) { data, response, error in
  if let error = error {
    print(error)
  }
}
dataTask.resume()
```
