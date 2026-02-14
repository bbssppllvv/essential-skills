# Create a Coinbase Charge for Crypto Payment

Create a Coinbase charge for cryptocurrency payment.

**Endpoint:** `POST https://openrouter.ai/api/v1/credits/coinbase`

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
| `amount` | number | Yes | Credit amount to charge |
| `sender` | string | Yes | Sender wallet address |
| `chain_id` | string | Yes | Blockchain network identifier. Enum: `1`, `137`, `8453` |

---

## Response

### 200 - Success

Returns calldata to fulfill the transaction.

**Response Body:**

```json
{
  "data": {
    "id": "string",
    "created_at": "string",
    "expires_at": "string",
    "web3_data": {
      "transfer_intent": {
        "call_data": {
          "deadline": "string",
          "fee_amount": "string",
          "id": "string",
          "operator": "string",
          "prefix": "string",
          "recipient": "string",
          "recipient_amount": "string",
          "recipient_currency": "string",
          "refund_destination": "string",
          "signature": "string"
        },
        "metadata": {
          "chain_id": 0,
          "contract_address": "string",
          "sender": "string"
        }
      }
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data.id` | string | Charge identifier |
| `data.created_at` | string | Creation timestamp |
| `data.expires_at` | string | Expiration timestamp |
| `data.web3_data.transfer_intent.call_data.deadline` | string | Transaction deadline |
| `data.web3_data.transfer_intent.call_data.fee_amount` | string | Fee amount |
| `data.web3_data.transfer_intent.call_data.id` | string | Call data identifier |
| `data.web3_data.transfer_intent.call_data.operator` | string | Operator address |
| `data.web3_data.transfer_intent.call_data.prefix` | string | Prefix |
| `data.web3_data.transfer_intent.call_data.recipient` | string | Recipient address |
| `data.web3_data.transfer_intent.call_data.recipient_amount` | string | Recipient amount |
| `data.web3_data.transfer_intent.call_data.recipient_currency` | string | Recipient currency |
| `data.web3_data.transfer_intent.call_data.refund_destination` | string | Refund destination address |
| `data.web3_data.transfer_intent.call_data.signature` | string | Transaction signature |
| `data.web3_data.transfer_intent.metadata.chain_id` | number | Blockchain network identifier |
| `data.web3_data.transfer_intent.metadata.contract_address` | string | Smart contract address |
| `data.web3_data.transfer_intent.metadata.sender` | string | Sender wallet address |

### Error Responses

| Status Code | Description |
|-------------|-------------|
| 400 | Bad Request - Invalid credit amount or request body |
| 401 | Unauthorized - Authentication required or invalid credentials |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Unexpected server error |

---

## Code Examples

### Python

```python
import requests

url = "https://openrouter.ai/api/v1/credits/coinbase"

payload = {
    "amount": 150,
    "sender": "0xAbC1234567890DefABC1234567890dEfABC12345",
    "chain_id": 1
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
const url = 'https://openrouter.ai/api/v1/credits/coinbase';
const options = {
  method: 'POST',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"amount":150,"sender":"0xAbC1234567890DefABC1234567890dEfABC12345","chain_id":1}'
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

	url := "https://openrouter.ai/api/v1/credits/coinbase"

	payload := strings.NewReader("{\n  \"amount\": 150,\n  \"sender\": \"0xAbC1234567890DefABC1234567890dEfABC12345\",\n  \"chain_id\": 1\n}")

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

url = URI("https://openrouter.ai/api/v1/credits/coinbase")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Post.new(url)
request["Authorization"] = 'Bearer <token>'
request["Content-Type"] = 'application/json'
request.body = "{\n  \"amount\": 150,\n  \"sender\": \"0xAbC1234567890DefABC1234567890dEfABC12345\",\n  \"chain_id\": 1\n}"

response = http.request(request)
puts response.read_body
```

### Java

```java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.Unirest;

HttpResponse<String> response = Unirest.post("https://openrouter.ai/api/v1/credits/coinbase")
  .header("Authorization", "Bearer <token>")
  .header("Content-Type", "application/json")
  .body("{\n  \"amount\": 150,\n  \"sender\": \"0xAbC1234567890DefABC1234567890dEfABC12345\",\n  \"chain_id\": 1\n}")
  .asString();
```

### PHP

```php
<?php
require_once('vendor/autoload.php');

$client = new \GuzzleHttp\Client();

$response = $client->request('POST', 'https://openrouter.ai/api/v1/credits/coinbase', [
  'body' => '{
  "amount": 150,
  "sender": "0xAbC1234567890DefABC1234567890dEfABC12345",
  "chain_id": 1
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

var client = new RestClient("https://openrouter.ai/api/v1/credits/coinbase");
var request = new RestRequest(Method.POST);
request.AddHeader("Authorization", "Bearer <token>");
request.AddHeader("Content-Type", "application/json");
request.AddParameter("application/json", "{\n  \"amount\": 150,\n  \"sender\": \"0xAbC1234567890DefABC1234567890dEfABC12345\",\n  \"chain_id\": 1\n}", ParameterType.RequestBody);
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
  "amount": 150,
  "sender": "0xAbC1234567890DefABC1234567890dEfABC12345",
  "chain_id": 1
] as [String : Any]

let postData = JSONSerialization.data(withJSONObject: parameters, options: [])

let request = NSMutableURLRequest(url: NSURL(string: "https://openrouter.ai/api/v1/credits/coinbase")! as URL,
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
