# Create a Message - OpenRouter API Documentation

## Overview

The Create a Message endpoint allows you to generate messages using the Anthropic Messages API format via OpenRouter. This endpoint supports multiple content types and advanced features.

**Endpoint:** `POST https://openrouter.ai/api/v1/messages`

**Content-Type:** `application/json`

## Supported Features

- Text and image content
- PDF document processing
- Tool integration and function calling
- Extended thinking capabilities
- Citations and content attribution
- Cache control mechanisms
- Web search functionality

## Authentication

The endpoint requires API key authentication via bearer token in the Authorization header.

## Request Schema

### Required Parameters

- **model** (string): The model identifier to use for generating the message
- **max_tokens** (number): Maximum tokens for the response
- **messages** (array): Array of message objects with role and content

### Message Structure

Each message requires:
- **role**: Either "user" or "assistant"
- **content**: Text string or array of content blocks (text, images, documents, tool usage, etc.)

### Optional Parameters

- **system**: System prompt (string or array of content blocks)
- **metadata**: Object with optional user_id field
- **stop_sequences**: Array of strings to stop generation
- **stream**: Boolean for streaming responses
- **temperature**: Float between 0 and 2
- **top_p**: Float for nucleus sampling
- **top_k**: Integer for top-k sampling
- **tools**: Array of available tools for the model
- **tool_choice**: Specification of tool usage preference
- **thinking**: Extended thinking configuration (enabled/disabled/adaptive)
- **service_tier**: "auto" or "standard_only"
- **plugins**: Array of plugins to enable
- **user**: End-user identifier (max 128 characters)
- **session_id**: Request grouping identifier (max 128 characters)
- **output_config**: Configuration for output effort level
- **provider**: Routing and provider preferences

## Content Types

### Text Content
Simple text blocks with optional citations and cache control.

### Image Content
Supports JPEG, PNG, GIF, and WebP formats via base64 encoding or URL.

### Document Content
PDF and plaintext documents with citation support and optional context.

### Tool Usage
Assistant-generated tool calls with ID, name, and input parameters.

### Tool Results
Tool execution results returned to the model for processing.

### Thinking Blocks
Extended thinking content with signature verification for reasoning tasks.

### Web Search Results
Integration with web search functionality for current information retrieval.

## Provider Configuration

Control routing behavior through the optional provider object:

- **allow_fallbacks**: Enable backup providers (default: true)
- **require_parameters**: Filter to parameter-supporting providers
- **data_collection**: "deny" or "allow" for data usage
- **zdr**: Restrict to Zero Data Retention endpoints
- **order**: Preferred provider sequence
- **only**: Whitelist specific providers
- **ignore**: Exclude specific providers
- **max_price**: Set cost limits per million tokens
- **preferred_min_throughput**: Minimum token throughput targets
- **preferred_max_latency**: Maximum latency preferences

## Plugins

Enable optional plugins for enhanced functionality:

- **auto-router**: Automatic model routing with optional filtering
- **moderation**: Content moderation checks
- **web**: Web search with configurable engine and result limits
- **file-parser**: Advanced PDF parsing with engine selection
- **response-healing**: Response correction and optimization

## Response Format

### Success Response (200)

Returns a message object containing:
- **type**: "message"
- **role**: "assistant"
- **content**: Array of content blocks (text, tool_use, thinking, etc.)
- **model**: The model used for generation
- **stop_reason**: Why generation stopped
- **usage**: Token count information

### Error Responses

- **400**: Invalid request parameters
- **401**: Authentication failure
- **403**: Permission denied
- **404**: Model or resource not found
- **429**: Rate limit exceeded
- **500**: API server error
- **503**: Service overloaded

## Tool Configuration

### Custom Tools
Define custom tools with JSON schema input specifications:

```json
{
  "type": "custom",
  "name": "tool_name",
  "description": "Tool description",
  "input_schema": {
    "type": "object",
    "properties": {},
    "required": []
  }
}
```

### Server Tools
Pre-built tools available through OpenRouter:
- **bash_20250124**: Execute bash commands
- **text_editor_20250124**: File editing operations
- **web_search_20250305**: Web search with location context

## Tool Choice Options

- **auto**: Model decides tool usage (with optional parallel disabling)
- **any**: Model must use a tool (with optional parallel disabling)
- **none**: Disable tool usage
- **tool**: Force specific named tool (with optional parallel disabling)

## Extended Thinking

Configure model reasoning with thinking configuration:

- **enabled**: Activate with specified token budget
- **disabled**: Turn off thinking
- **adaptive**: Let model decide when to think

## Output Configuration

Optionally specify effort level for response generation:
- low, medium, high, max
