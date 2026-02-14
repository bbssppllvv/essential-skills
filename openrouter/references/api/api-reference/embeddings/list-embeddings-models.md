# List All Embeddings Models

## Endpoint Overview

The `/embeddings/models` endpoint enables retrieval of available embeddings models and their specifications.

**Request Method:** GET
**URL:** `https://openrouter.ai/api/v1/embeddings/models`

## Purpose

This endpoint "Returns a list of all available embeddings models and their properties"

## Authentication

The request requires an API key passed as a bearer token in the Authorization header.

## Response Structure

The API returns a `ModelsListResponse` object containing:

- **data**: An array of Model objects, each including:
  - `id`: Unique model identifier
  - `name`: Display name
  - `canonical_slug`: Model slug
  - `description`: Model details
  - `pricing`: Cost information (prompt, completion, requests, images, audio, caching, web search, reasoning)
  - `context_length`: Token limit
  - `architecture`: Tokenizer, instruction format, modalities
  - `top_provider`: Context details and moderation status
  - `supported_parameters`: Available configuration options
  - `default_parameters`: Preset values
  - `expiration_date`: Removal timeline (if applicable)

## Supported HTTP Status Codes

- **200**: Successful response with models list
- **400**: Invalid request parameters
- **500**: Server error

## Code Examples

### Python
```python
import requests
url = "https://openrouter.ai/api/v1/embeddings/models"
headers = {"Authorization": "Bearer <token>"}
response = requests.get(url, headers=headers)
print(response.json())
```

### JavaScript
```javascript
const url = 'https://openrouter.ai/api/v1/embeddings/models';
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
import ("fmt"; "net/http"; "io")
func main() {
  url := "https://openrouter.ai/api/v1/embeddings/models"
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
url = URI("https://openrouter.ai/api/v1/embeddings/models")
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
HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/embeddings/models")
  .header("Authorization", "Bearer <token>")
  .asString();
```

### PHP
```php
<?php
require_once('vendor/autoload.php');
$client = new \GuzzleHttp\Client();
$response = $client->request('GET', 'https://openrouter.ai/api/v1/embeddings/models', [
  'headers' => ['Authorization' => 'Bearer <token>'],
]);
echo $response->getBody();
```

### C#
```csharp
using RestSharp;
var client = new RestClient("https://openrouter.ai/api/v1/embeddings/models");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

### Swift
```swift
import Foundation
let headers = ["Authorization": "Bearer <token>"]
let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/embeddings/models")! as URL)
request.httpMethod = "GET"
request.allHTTPHeaderFields = headers
let session = URLSession.shared
let dataTask = session.dataTask(with: request as URLRequest)
dataTask.resume()
```
