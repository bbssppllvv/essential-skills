# Update a Guardrail API Documentation

## Endpoint Overview

The Update a Guardrail endpoint enables modifications to existing guardrails through a PATCH request to `https://openrouter.ai/api/v1/guardrails/{id}`. This operation requires a Management API key for authentication.

## Request Details

**HTTP Method:** PATCH
**Authentication:** Bearer token in Authorization header (Management key required)
**Content-Type:** application/json

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | UUID string | The unique identifier of the guardrail to update |

### Request Body Properties

- **name** (string): New guardrail name
- **description** (string or null): Updated description
- **limit_usd** (number or null): Spending limit in USD
- **reset_interval** (string or null): Limit reset frequency -- options include daily, weekly, or monthly
- **allowed_providers** (array or null): List of provider IDs
- **allowed_models** (array or null): Model identifiers (slug or canonical_slug)
- **enforce_zdr** (boolean or null): Whether to enforce zero data retention

## Response Structure

A successful 200 response returns an object containing:

- **id** (UUID): Guardrail identifier
- **name** (string): Guardrail name
- **description** (string or null): Description text
- **limit_usd** (number or null): Spending cap
- **reset_interval** (string or null): Reset frequency
- **allowed_providers** (array or null): Provider list
- **allowed_models** (array or null): Array of model canonical_slugs (immutable identifiers)
- **enforce_zdr** (boolean or null): Zero data retention setting
- **created_at** (ISO 8601 timestamp): Creation time
- **updated_at** (ISO 8601 timestamp or null): Last modification time

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Guardrail updated successfully |
| 400 | Invalid request parameters |
| 401 | Missing or invalid authentication |
| 404 | Guardrail does not exist |
| 500 | Server error |

## Code Examples

**Python:**
```python
import requests

url = "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000"
payload = {
    "name": "Updated Guardrail Name",
    "description": "Updated description",
    "limit_usd": 75,
    "reset_interval": "weekly"
}
headers = {
    "Authorization": "Bearer <token>",
    "Content-Type": "application/json"
}

response = requests.patch(url, json=payload, headers=headers)
print(response.json())
```

**JavaScript:**
```javascript
const url = 'https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000';
const options = {
  method: 'PATCH',
  headers: {Authorization: 'Bearer <token>', 'Content-Type': 'application/json'},
  body: '{"name":"Updated Guardrail Name","description":"Updated description","limit_usd":75,"reset_interval":"weekly"}'
};

fetch(url, options)
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

**Go:**
```go
package main

import (
	"fmt"
	"strings"
	"net/http"
	"io"
)

func main() {
	url := "https://openrouter.ai/api/v1/guardrails/550e8400-e29b-41d4-a716-446655440000"
	payload := strings.NewReader(`{"name":"Updated Guardrail Name","description":"Updated description","limit_usd":75,"reset_interval":"weekly"}`)

	req, _ := http.NewRequest("PATCH", url, payload)
	req.Header.Add("Authorization", "Bearer <token>")
	req.Header.Add("Content-Type", "application/json")

	res, _ := http.DefaultClient.Do(req)
	defer res.Body.Close()
	body, _ := io.ReadAll(res.Body)

	fmt.Println(string(body))
}
```

Additional code examples are provided for Ruby, Java, PHP, C#, and Swift using similar patterns with language-specific HTTP clients.
