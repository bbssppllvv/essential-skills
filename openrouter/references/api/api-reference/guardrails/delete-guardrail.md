# Delete a Guardrail

`DELETE https://openrouter.ai/api/v1/guardrails/{id}`

Delete a guardrail by ID. Requires a management API key.

## Authentication

Requires a Management API key passed as a bearer token in the `Authorization` header.

## Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | string (uuid) | Yes | The unique identifier of the guardrail to delete |

## Response (200 OK)

Returns a JSON object confirming deletion:

```json
{
  "deleted": true
}
```

## Error Responses

| Code | Description |
|------|-------------|
| 401 | Unauthorized - Missing or invalid authentication |
| 404 | Not Found - Guardrail does not exist |
| 500 | Internal Server Error |

## Code Examples

### Python

```python
import requests
url = "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000"
headers = {"Authorization": "Bearer <token>"}
response = requests.delete(url, headers=headers)
print(response.json())
```

### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000';
const options = {method: 'DELETE', headers: {Authorization: 'Bearer <token>'}};
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
import ("fmt"; "net/http"; "io")
func main() {
	url := "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000"
	req, _ := http.NewRequest("DELETE", url, nil)
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
url = URI("https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000")
http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true
request = Net::HTTP::Delete.new(url)
request["Authorization"] = 'Bearer <token>'
response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;
HttpResponse<String> response = Unirest.delete("https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');
$client = new \GuzzleHttp\Client();
$response = $client->request('DELETE', 'https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000', [
  'headers' => ['Authorization' => 'Bearer <token>'],
]);
echo $response->getBody();
```

### C#

```csharp
using RestSharp;
var client = new RestClient("https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000");
var request = new RestRequest(Method.DELETE);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift

```swift
import Foundation
let headers = ["Authorization": "Bearer <token>"]
let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000")! as URL, cachePolicy: .useProtocolCachePolicy, timeoutInterval: 10.0)
request.httpMethod = "DELETE"
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
