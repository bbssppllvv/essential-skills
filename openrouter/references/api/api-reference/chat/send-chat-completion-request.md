# OpenRouter Chat Completions API Documentation

## Endpoint Overview

The OpenRouter Chat Completions API enables developers to send chat requests to various language models through a unified interface.

**Endpoint:** `POST https://openrouter.ai/api/v1/chat/completions`

**Content-Type:** `application/json`

## Core Functionality

This endpoint sends a request for a model response for the given chat conversation and supports both streaming and non-streaming response modes.

## Authentication

API requests require Bearer token authentication via the Authorization header.

## Request Parameters

### Essential Parameters

- **messages** (required): Array of message objects representing the conversation history
- **model**: Identifier for the language model to use

### Message Structure

Supported message roles include:
- `system`: Provides context or instructions
- `user`: User input
- `developer`: Development-level instructions
- `assistant`: Model responses
- `tool`: Tool/function responses

Message content can be text or multimodal (images, audio, video).

### Response Configuration

- **response_format**: Supports text, JSON objects, JSON Schema, grammar, or Python output
- **stream**: Boolean flag enabling streaming responses (default: false)
- **stream_options**: Configure streaming behavior (e.g., include usage statistics)

### Generation Parameters

- **temperature**: Controls response randomness (default: 1)
- **top_p**: Nucleus sampling parameter (default: 1)
- **max_tokens** / **max_completion_tokens**: Output length limits
- **frequency_penalty** / **presence_penalty**: Repetition controls
- **stop**: Sequence(s) that terminate generation
- **seed**: For reproducible outputs
- **logprobs** / **top_logprobs**: Token probability data

### Tool Integration

- **tools**: Array of function definitions
- **tool_choice**: Specify tool usage behavior (auto, none, required, or specific function)

### Advanced Features

- **reasoning**: Configure reasoning effort and summary verbosity
- **plugins**: Enable features like web search, file parsing, and response healing
- **provider**: Route requests through specific providers or customize provider selection

### Observability

- **session_id**: Groups related requests for tracking
- **trace**: Metadata including trace_id, span_name, and custom fields
- **user**: User identifier for request tracking

## Response Structure

### Success Response (200)

Returns a ChatResponse object containing:

- **id**: Unique request identifier
- **choices**: Array of completion choices with:
  - **message**: Assistant's response (with role, content, optional tool_calls)
  - **finish_reason**: Completion status (stop, tool_calls, length, content_filter, error)
  - **logprobs**: Token-level probability data (when requested)
- **usage**: Token count details including prompt/completion/total tokens
- **created**: Timestamp
- **model**: Model identifier used
- **object**: Response type ("chat.completion")

### Error Responses

- **400**: Invalid parameters
- **401**: Authentication failure
- **429**: Rate limit exceeded
- **500**: Server error

## SDK Code Examples

The documentation provides complete implementation examples in:
- Python
- JavaScript
- Go
- Ruby
- Java
- PHP
- C#
- Swift

All examples demonstrate the basic pattern: POST request with JSON payload and Authorization header.

## Provider Management

The API supports sophisticated provider selection through:
- **order**: Prioritized list of provider slugs
- **only**: Allowlist of providers
- **ignore**: Provider blocklist
- **sort**: Route by price, throughput, or latency
- **allow_fallbacks**: Enable backup provider fallback
- **data_collection**: Filter providers by data handling policies

## Additional Features

- **Quantization filtering**: Target specific model quantization levels
- **Price controls**: Set maximum acceptable pricing per million tokens
- **Performance preferences**: Specify minimum throughput or maximum latency thresholds
- **Multimodal support**: Handle text, images, audio, and video inputs
