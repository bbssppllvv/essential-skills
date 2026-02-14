# List All Models and Their Properties

## Endpoint Overview

The OpenRouter API provides a GET endpoint at `https://openrouter.ai/api/v1/models` that retrieves available models with their properties.

## Request Details

**Method:** GET
**Authentication:** Bearer token required in Authorization header

### Query Parameters

- **category** (optional): Filter by use case category (programming, roleplay, marketing, marketing/seo, technology, science, translation, legal, finance, health, trivia, academia)
- **supported_parameters** (optional): String parameter for filtering
- **use_rss** (optional): String parameter
- **use_rss_chat_links** (optional): String parameter

## Response Structure

The endpoint returns a JSON object containing a `data` array of model objects. Each model includes:

- **id**: Unique identifier
- **canonical_slug**: URL-friendly model name
- **name**: Display name
- **created**: Unix timestamp
- **description**: Model overview
- **pricing**: Detailed pricing structure (prompt, completion, request, image, audio, web search, etc.)
- **context_length**: Maximum tokens supported
- **architecture**: Tokenizer, instruction type, and modality information
- **top_provider**: Context limits and moderation status
- **per_request_limits**: Prompt and completion token restrictions
- **supported_parameters**: Available configuration options
- **default_parameters**: Temperature, top_p, frequency penalty defaults
- **expiration_date**: ISO 8601 date or null

## HTTP Status Codes

- **200**: Successful response with model list
- **400**: Invalid request parameters
- **500**: Server error

## Code Examples

Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift implementations are provided, all following the same pattern: GET request with Bearer token authorization header.
