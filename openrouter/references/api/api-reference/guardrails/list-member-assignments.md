# List All Member Guardrail Assignments

`GET https://openrouter.ai/api/v1/guardrails/assignments/members`

List all organization member guardrail assignments for the authenticated user. Requires a management API key.

## Authentication

Requires a Management API key passed as a bearer token in the `Authorization` header.

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| offset | string | No | Number of records to skip for pagination |
| limit | string | No | Maximum number of records to return (max 100) |

## Response (200 OK)

| Field | Type | Description |
|-------|------|-------------|
| data | array | List of member assignment objects |
| total_count | number | Total number of member assignments |

### Member Assignment Object

| Field | Type | Description |
|-------|------|-------------|
| id | string (uuid) | Unique identifier for the assignment |
| user_id | string | Clerk user ID of the assigned member |
| organization_id | string | Organization ID |
| guardrail_id | string (uuid) | ID of the assigned guardrail |
| assigned_by | string or null | User ID who made the assignment |
| created_at | string | ISO 8601 timestamp of when the assignment was created |

## Error Responses

| Code | Description |
|------|-------------|
| 401 | Unauthorized - Missing or invalid authentication |
| 500 | Internal Server Error |

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/guardrails/assignments/members"
headers = {"Authorization": "Bearer <token>"}
response = requests.get(url, headers=headers)
print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/guardrails/assignments/members';
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
	url := "https://openrouter.ai/api/v1/guardrails/assignments/members"
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

url = URI("https://openrouter.ai/api/v1/guardrails/assignments/members")
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

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/guardrails/assignments/members")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();
$response = $client->request('GET', 'https://openrouter.ai/api/v1/guardrails/assignments/members', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);
echo $response->getBody();
```

### C#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/guardrails/assignments/members");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]
let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/guardrails/assignments/members")! as URL,
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
