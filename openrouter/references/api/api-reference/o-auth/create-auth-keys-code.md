# Create Authorization Code

Create an authorization code for the PKCE flow to generate a user-controlled API key.

**Endpoint:** `POST https://openrouter.ai/api/v1/auth/keys/code`

**Content-Type:** application/json

**Authentication:** Requires an API key passed as a Bearer token in the Authorization header.

---

## Headers

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `Authorization` | string | Yes | API key as bearer token in Authorization header |
| `Content-Type` | string | Yes | `application/json` |

---

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `callback_url` | string (URI) | Yes | The callback URL to redirect to after authorization. Note, only https URLs on ports 443 and 3000 are allowed. |
| `code_challenge` | string | Yes | PKCE code challenge for enhanced security |
| `code_challenge_method` | string | Yes | Method for code challenge generation. Enum: `S256`, `plain` |
| `limit` | number | No | Credit limit for the API key |
| `expires_at` | string (date-time) or null | No | Optional expiration time (ISO 8601 format) |

---

## Response

### 200 - Success

Returns the created authorization code data.

**Response Body:**

```json
{
  "data": {
    "id": "string",
    "app_id": 0,
    "created_at": "string"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data.id` | string | The authorization code ID to use in the exchange request |
| `data.app_id` | number | Application ID associated with the authorization code |
| `data.created_at` | string | ISO 8601 timestamp of creation |

### Error Responses

| Status Code | Description |
|-------------|-------------|
| 400 | Bad Request - Invalid parameters or malformed input |
| 401 | Unauthorized - Authentication required or invalid credentials |
| 500 | Internal Server Error - Unexpected server error |

---

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/auth/keys/code"

payload = {
    "callback_url": "https://myapp.com/auth/callback",
    "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM",
    "code_challenge_method": "S256",
    "limit": 100
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
const url = 'https://openrouter.ai/api/v1/auth/keys/code';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"callback_url":"https://myapp.com/auth/callback","code_challenge":"E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM","code_challenge_method":"S256","limit":100}'
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

	url := "https://openrouter.ai/api/v1/auth/keys/code"

	payload := strings.NewReader("{\n  \"callback_url\": \"https://myapp.com/auth/callback\",\n  \"code_challenge\": \"E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM\",\n  \"code_challenge_method\": \"S256\",\n  \"limit\": 100\n}")

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

url = URI("https://openrouter.ai/api/v1/auth/keys/code")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"callback_url\": \"https://myapp.com/auth/callback\",\n  \"code_challenge\": \"E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM\",\n  \"code_challenge_method\": \"S256\",\n  \"limit\": 100\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/auth/keys/code")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"callback_url\": \"https://myapp.com/auth/callback\",\n  \"code_challenge\": \"E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM\",\n  \"code_challenge_method\": \"S256\",\n  \"limit\": 100\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/auth/keys/code', [
  'body' => '{
  "callback_url": "https://myapp.com/auth/callback",
  "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM",
  "code_challenge_method": "S256",
  "limit": 100
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

var client = new RestClient("https://openrouter.ai/api/v1/auth/keys/code");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"callback_url\": \"https://myapp.com/auth/callback\",\n  \"code_challenge\": \"E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM\",\n  \"code_challenge_method\": \"S256\",\n  \"limit\": 100\n}", ParameterType.RequestBody);
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
  "callback_url": "https://myapp.com/auth/callback",
  "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM",
  "code_challenge_method": "S256",
  "limit": 100
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/auth/keys/code")! as URL,
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
