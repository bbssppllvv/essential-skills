# Web Search Documentation

## Overview
OpenRouter's Responses API Beta includes web search capabilities for accessing real-time information with citation annotations. The feature is currently in beta and may undergo breaking changes.

## Key Features

**Web Search Plugin Configuration:**
The implementation uses a simple plugin system with two parameters:
- `id`: Must be set to "web" (required)
- `max_results`: Accepts integers between 1-10

## Implementation Methods

The documentation outlines three approaches:

1. **Simple String Query**: Direct input with plugin parameters
2. **Structured Messages**: Complex queries using message format with content arrays
3. **Online Model Variants**: Using the `:online` suffix (e.g., `openai/o4-mini:online`) for models with built-in search

## Response Structure

Responses include citation annotations that reference source URLs. The structure contains:
- Message output with text content
- Annotations array tracking source citations
- Start/end indices mapping citations to specific text passages

## Advanced Capabilities

The API supports:
- Multi-turn conversations with search enabled
- Streaming responses with real-time progress monitoring
- Citation extraction and processing
- Complex multi-part queries

## Recommended Practices

- Balance result quantity with response speed
- Process citations properly for attribution
- Craft specific search queries
- Implement error handling for search failures
- Respect rate limiting thresholds
