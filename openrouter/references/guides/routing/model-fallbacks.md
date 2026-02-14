# Model Fallbacks Documentation

## Overview

OpenRouter's model fallbacks feature enables "automatic failover between AI models when providers are down, rate-limited, or refuse requests."

## Core Functionality

The `models` parameter accepts an array of model IDs ordered by preference. When the primary model encounters an error, OpenRouter automatically attempts the next model in the sequence.

## Fallback Triggers

The system activates fallback behavior for multiple error types:

- Context length validation failures
- Content moderation flags on filtered models
- Rate-limiting constraints
- Provider downtime

## Pricing Model

"Requests are priced using the model that was ultimately used, which will be returned in the `model` attribute of the response body."

## Implementation Examples

**OpenRouter SDK (TypeScript):**
```typescript
const completion = await openRouter.chat.send({
  models: ['anthropic/claude-3.5-sonnet', 'gryphe/mythomax-l2-13b'],
  messages: [{ role: 'user', content: 'Your prompt here' }],
});
```

**REST API (curl/fetch):**
Include the `models` array in the JSON request body to the `/api/v1/chat/completions` endpoint.

**Python:**
Use the `requests` library to POST JSON data containing the models array to the OpenRouter API endpoint.

## OpenAI SDK Integration

When using OpenAI's SDK, pass the models array through the `extra_body` parameter. The specified `model` parameter acts as the primary option, with array entries serving as sequential fallbacks.
