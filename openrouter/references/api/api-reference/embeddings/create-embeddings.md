# OpenRouter Embeddings API Documentation

## Endpoint Overview

The OpenRouter embeddings API allows you to submit embedding requests through a unified router interface.

**Endpoint:** `POST https://openrouter.ai/api/v1/embeddings`

## Required Parameters

- **input**: The text or content to embed (supports multiple formats)
- **model**: The embedding model identifier
- **Authorization**: Bearer token in header (required)

## Input Formats

The `input` parameter accepts several formats:
- Single string: `"text here"`
- Array of strings: `["text1", "text2"]`
- Array of numbers (embeddings): `[[0.1, 0.2], [0.3, 0.4]]`
- Complex content with text and image URLs

## Optional Parameters

- **encoding_format**: Output format (`float` or `base64`)
- **dimensions**: Desired embedding dimensions
- **user**: User identifier string
- **input_type**: Specification for input content type
- **provider**: Routing preferences and provider constraints

## Response Structure

Successful responses (HTTP 200) include:
- `id`: Request identifier
- `object`: Always "list"
- `data`: Array of embedding objects with `embedding`, `object`, and `index`
- `model`: Model used
- `usage`: Token counts and cost information

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Embedding response successful |
| 400 | Invalid request parameters |
| 401 | Authentication required or invalid |
| 402 | Insufficient credits/quota |
| 404 | Resource not found |
| 429 | Rate limit exceeded |
| 500-503 | Server errors |

## Code Examples

Multiple language implementations are provided, including Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift. All examples follow the same pattern: POST request with JSON payload containing `input` and `model` parameters, authenticated via Bearer token.
