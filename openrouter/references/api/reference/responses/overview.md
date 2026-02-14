# OpenRouter Responses API Beta Documentation

## Overview

OpenRouter provides a beta-stage API that mirrors OpenAI's Responses API structure while supporting multiple AI models through a unified interface. The service operates as a stateless transformation layer with support for reasoning, tool calling, and web search.

## Key Characteristics

**Stateless Architecture**: Each API call operates independently without persisting conversation history on the server. Users must supply the complete message thread with every request.

**Beta Status**: The API is under active development and may introduce breaking changes, requiring caution in production deployments.

## Technical Details

**Base Endpoint**: `https://openrouter.ai/api/v1/responses`

**Authentication Method**: Bearer token authentication using your OpenRouter API key in the Authorization header

## Core Capabilities

The platform enables four primary functionality areas:

1. Basic text input/output operations
2. Advanced reasoning with configurable effort settings
3. Function calling with parallel execution support
4. Real-time web search with citation tracking

## Error Response Format

The API returns structured errors containing a code identifier, descriptive message, and metadata field:

```json
{
  "error": {
    "code": "invalid_prompt",
    "message": "Missing required parameter: 'model'."
  },
  "metadata": null
}
```

## Rate Limiting

Standard OpenRouter rate limits govern API usage, documented separately in the API reference section.
