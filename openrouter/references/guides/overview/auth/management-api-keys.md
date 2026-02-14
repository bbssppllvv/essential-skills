# Management API Keys

Programmatic Control of OpenRouter API Keys

---

OpenRouter provides endpoints to programmatically manage your API keys, enabling key creation and management for applications that need to distribute or rotate keys automatically.

> **Important**: Management keys cannot be used to make API calls to OpenRouter's completion endpoints - they are exclusively for administrative operations.

## Common Use Cases

Common scenarios for programmatic key management include:

- **SaaS Applications**: Automatically create unique API keys for each customer instance
- **Key Rotation**: Regularly rotate API keys for security compliance
- **Usage Monitoring**: Track key usage and automatically disable keys that exceed limits

## API Endpoints

All key management endpoints are under `/api/v1/keys` and require a Management API key in the `Authorization` header.

---

## Create a New API Key

**Endpoint:** `POST https://openrouter.ai/api/v1/keys`

**Authentication:** Requires management API key as bearer token

Creates a new API key for the authenticated user with optional spending limits and expiration settings.

### Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Identifier for the new key |
| `limit` | number \| null | No | Spending cap in USD |
| `limit_reset` | string \| null | No | Reset frequency: `daily`, `weekly`, `monthly` |
| `include_byok_in_limit` | boolean | No | Count external BYOK usage toward limit |
| `expires_at` | ISO 8601 timestamp \| null | No | Expiration date in UTC format |

> **Note:** The type of limit reset for the API key can be `daily`, `weekly`, `monthly`, or `null` for no reset. Resets happen automatically at midnight UTC, and weeks are Monday through Sunday.

### Response (201 Success)

Returns an object containing:

- `key`: The actual API key string (displayed only once)
- `data`: Complete key metadata including hash, creation timestamp, usage metrics (total, daily, weekly, monthly for both standard and BYOK usage), disabled status, and expiration details

### Error Responses

| Code | Description |
|------|-------------|
| 400 | Invalid request parameters |
| 401 | Missing/invalid authentication |
| 429 | Rate limit exceeded |
| 500 | Server error |

### Code Examples

#### Python

```python
import requests

url = "https://openrouter.ai/api/v1/keys"

payload = {
    "name": "Analytics Service Key",
    "limit": 150,
    "limit_reset": "monthly",
    "include_byok_in_limit": True,
    "expires_at": "2028-06-30T23:59:59Z"
}
headers = {
    "Authorization": "Bearer <token>",
    "Content-Type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)

print(response.json())
```

#### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/keys';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"name":"Analytics Service Key","limit":150,"limit_reset":"monthly","include_byok_in_limit":true,"expires_at":"2028-06-30T23:59:59Z"}'
};

try {
  const response = await fetch(url, options);
  const data = await response.json();
  console.log(data);
} catch (error) {
  console.error(error);
}
```

#### Go

```go
package main

import (
	"fmt"
	"strings"
	"net/http"
	"io"
)

func main() {

	url := "https://openrouter.ai/api/v1/keys"

	payload := strings.NewReader("{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}")

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

#### Ruby

```ruby
require 'uri'
require 'net/http'

url = URI("https://openrouter.ai/api/v1/keys")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}"

response = http.request(request)
puts response.read_body
```

#### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/keys")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}")
  .asString();
```

#### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/keys', [
  'body' => '{
  "name": "Analytics Service Key",
  "limit": 150,
  "limit_reset": "monthly",
  "include_byok_in_limit": true,
  "expires_at": "2028-06-30T23:59:59Z"
}',
  'headers' => [
    'Authorization' => 'Bearer <token>',
    'Content-Type' => 'application/json',
  ],
]);

echo $response->getBody();
```

#### C\#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/keys");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"name\": \"Analytics Service Key\",\n  \"limit\": 150,\n  \"limit_reset\": \"monthly\",\n  \"include_byok_in_limit\": true,\n  \"expires_at\": \"2028-06-30T23:59:59Z\"\n}", ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

#### Swift

```swift
import Foundation

let headers = [
  "Authorization": "Bearer <token>",
  "Content-Type": "application/json"
]
let parameters = [
  "name": "Analytics Service Key",
  "limit": 150,
  "limit_reset": "monthly",
  "include_byok_in_limit": true,
  "expires_at": "2028-06-30T23:59:59Z"
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/keys")! as URL,
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

---

## List API Keys

**Endpoint:** `GET https://openrouter.ai/api/v1/keys`

**Authentication:** Requires management API key as bearer token

Retrieves all API keys for the authenticated user.

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `include_disabled` | boolean | No | Include disabled keys in results |
| `offset` | number | No | Pagination parameter to skip keys |

### Response (200 Success)

Returns a `data` array with API key objects including:

- `hash`: Unique identifier
- `name` and `label`: Key naming information
- `disabled`: Boolean status flag
- `limit` and `limit_remaining`: Spending limits in USD
- `limit_reset`: Reset schedule type
- `include_byok_in_limit`: BYOK inclusion setting
- `usage`, `usage_daily`, `usage_weekly`, `usage_monthly`: Credit tracking
- `byok_usage`, `byok_usage_daily`, `byok_usage_weekly`, `byok_usage_monthly`: External BYOK usage metrics
- `created_at`, `updated_at`, `expires_at`: Timestamps

### Error Responses

| Code | Description |
|------|-------------|
| 401 | Authentication failure |
| 429 | Rate limit exceeded |
| 500 | Server error |

### Code Examples

#### Python

```python
import requests

url = "https://openrouter.ai/api/v1/keys"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

#### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/keys';
const options = {method: 'GET', headers: {Authorization: 'Bearer <token>'}};

try {
  const response = await fetch(url, options);
  const data = await response.json();
  console.log(data);
} catch (error) {
  console.error(error);
}
```

#### Go

```go
package main

import (
	"fmt"
	"net/http"
	"io"
)

func main() {

	url := "https://openrouter.ai/api/v1/keys"

	req, _ := http.NewRequest("GET", url, nil)

	req.Header.Add("Authorization", "Bearer <token>")

	res, _ := http.DefaultClient.Do(req)

	defer res.Body.Close()
	body, _ := io.ReadAll(res.Body)

	fmt.Println(res)
	fmt.Println(string(body))

}
```

#### Ruby

```ruby
require 'uri'
require 'net/http'

url = URI("https://openrouter.ai/api/v1/keys")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Get.new(url)
request["Authorization"] = 'Bearer <token>'

response = http.request(request)
puts response.read_body
```

#### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/keys")
  .header("Authorization", "Bearer <token>")
  .asString();
```

#### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('GET', 'https://openrouter.ai/api/v1/keys', [
  'headers' => [
    'Authorization' => 'Bearer <token>',
  ],
]);

echo $response->getBody();
```

#### C\#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/keys");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

#### Swift

```swift
import Foundation

let headers = ["Authorization": "Bearer <token>"]

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/keys")! as URL,
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

---

## Get a Single API Key

**Endpoint:** `GET https://openrouter.ai/api/v1/keys/{hash}`

**Authentication:** Requires management API key as bearer token

Retrieves details for a specific API key using its hash identifier.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hash` | string | Yes | The hash identifier of the API key to retrieve |

### Response (200 Success)

Returns a `data` object containing:

- `hash`: Unique identifier
- `name`: Key name
- `label`: Human-readable designation
- `disabled`: Active/inactive status
- `limit`: Spending cap in USD
- `limit_remaining`: Available budget in USD
- `limit_reset`: Reset schedule type
- `include_byok_in_limit`: BYOK inclusion setting
- `usage`: Total OpenRouter spending
- `usage_daily`, `usage_weekly`, `usage_monthly`: Period-specific spending
- `byok_usage`: External provider expenses
- `byok_usage_daily`, `byok_usage_weekly`, `byok_usage_monthly`: Period-specific external costs
- `created_at`: Creation timestamp (ISO 8601)
- `updated_at`: Last modification timestamp (ISO 8601, may be null)
- `expires_at`: Expiration timestamp or null

### Error Responses

| Code | Description |
|------|-------------|
| 401 | Missing or invalid authentication |
| 404 | API key does not exist |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

### Code Examples

#### Python

```python
import requests

url = "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943"

headers = {"Authorization": "Bearer <token>"}

response = requests.get(url, headers=headers)

print(response.json())
```

#### JavaScript

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

#### Go

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

#### Ruby

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

#### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.get("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")
  .header("Authorization", "Bearer <token>")
  .asString();
```

#### PHP

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

#### C\#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943");
var request = new RestRequest(Method.GET);
request.AddHeader("Authorization", "Bearer <token>");
IRestResponse response = client.Execute(request);
```

#### Swift

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

---

## Update an API Key

**Endpoint:** `PATCH https://openrouter.ai/api/v1/keys/{hash}`

**Authentication:** Requires management API key as bearer token

Modifies an existing API key's properties.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hash` | string | Yes | Unique identifier for the API key being updated |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | New identifier for the API key |
| `disabled` | boolean | No | Toggle to deactivate the key |
| `limit` | number \| null | No | Spending cap in USD |
| `limit_reset` | string \| null | No | Reset frequency: `daily`, `weekly`, `monthly`, or `null` |
| `include_byok_in_limit` | boolean | No | Whether to count external BYOK expenses toward the limit |

### Response (200 Success)

Returns an object containing:

- `hash`: Unique identifier
- `name`: Current name
- `label`: Human-readable designation
- `disabled`: Active/inactive status
- `limit`: Spending ceiling in USD
- `limit_remaining`: Available budget in USD
- `limit_reset`: Reset schedule type
- `include_byok_in_limit`: BYOK inclusion setting
- `usage`: Total OpenRouter spending
- `usage_daily`, `usage_weekly`, `usage_monthly`: Period-specific spending
- `byok_usage`: External provider expenses
- `byok_usage_daily`, `byok_usage_weekly`, `byok_usage_monthly`: Period-specific external costs
- `created_at`: Creation timestamp (ISO 8601)
- `updated_at`: Last modification timestamp (ISO 8601)
- `expires_at`: Expiration timestamp or null

### Error Responses

| Code | Description |
|------|-------------|
| 400 | Invalid parameters |
| 401 | Authentication failure |
| 404 | Key not found |
| 429 | Rate limit exceeded |
| 500 | Server error |

### Code Examples

#### Python

```python
import requests

url = "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943"

payload = {
    "name": "Updated API Key Name",
    "disabled": False,
    "limit": 75,
    "limit_reset": "daily",
    "include_byok_in_limit": True
}
headers = {
    "Authorization": "Bearer <token>",
    "Content-Type": "application/json"
}

response = requests.patch(url, json=payload, headers=headers)

print(response.json())
```

#### JavaScript

```javascript
const url = 'https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943';
const options = {
  method: 'PATCH',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"name":"Updated API Key Name","disabled":false,"limit":75,"limit_reset":"daily","include_byok_in_limit":true}'
};

try {
  const response = await fetch(url, options);
  const data = await response.json();
  console.log(data);
} catch (error) {
  console.error(error);
}
```

#### Go

```go
package main

import (
	"fmt"
	"strings"
	"net/http"
	"io"
)

func main() {

	url := "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943"

	payload := strings.NewReader("{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}")

	req, _ := http.NewRequest("PATCH", url, payload)

	req.Header.Add("Authorization", "Bearer <token>")
	req.Header.Add("Content-Type", "application/json")

	res, _ := http.DefaultClient.Do(req)

	defer res.Body.Close()
	body, _ := io.ReadAll(res.Body)

	fmt.Println(res)
	fmt.Println(string(body))

}
```

#### Ruby

```ruby
require 'uri'
require 'net/http'

url = URI("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Patch.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}"

response = http.request(request)
puts response.read_body
```

#### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.patch("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}")
  .asString();
```

#### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('PATCH', 'https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943', [
  'body' => '{
  "name": "Updated API Key Name",
  "disabled": false,
  "limit": 75,
  "limit_reset": "daily",
  "include_byok_in_limit": true
}',
  'headers' => [
    'Authorization' => 'Bearer <token>',
    'Content-Type' => 'application/json',
  ],
]);

echo $response->getBody();
```

#### C\#

```csharp
using RestSharp;

var client = new RestClient("https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943");
var request = new RestRequest(Method.PATCH);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"name\": \"Updated API Key Name\",\n  \"disabled\": false,\n  \"limit\": 75,\n  \"limit_reset\": \"daily\",\n  \"include_byok_in_limit\": true\n}", ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

#### Swift

```swift
import Foundation

let headers = [
  "Authorization": "Bearer <token>",
  "Content-Type": "application/json"
]
let parameters = [
  "name": "Updated API Key Name",
  "disabled": false,
  "limit": 75,
  "limit_reset": "daily",
  "include_byok_in_limit": true
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/keys/f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943")! as URL,
                                        cachePolicy: .useProtocolCachePolicy,
                                    timeoutInterval: 10.0)
request.httpMethod = "PATCH"
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

---

## Delete an API Key

**Endpoint:** `DELETE https://openrouter.ai/api/v1/keys/{hash}`

**Authentication:** Requires management API key as bearer token

Permanently deletes an API key identified by its hash.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hash` | string | Yes | The hash identifier of the API key to delete |

### Error Responses

| Code | Description |
|------|-------------|
| 401 | Authentication failure |
| 404 | Key not found |
| 429 | Rate limit exceeded |
| 500 | Server error |

---

## Related Documentation

- [API Key Rotation](/docs/guides/guides/api-key-rotation) - Best practices for rotating keys
- [API Authentication](/docs/api/reference/authentication) - Authentication methods overview
- [Enterprise Quickstart](/docs/enterprise-quickstart) - Organization key management
