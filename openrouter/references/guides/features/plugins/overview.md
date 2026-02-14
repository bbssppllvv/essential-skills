# Plugins Documentation

## Overview

OpenRouter provides plugins to enhance model capabilities with features including real-time web search, PDF processing, and automatic JSON repair. These can be enabled per-request via API or configured as account defaults through the Plugins settings page.

## Available Plugins

| Plugin | Purpose | Documentation |
|--------|---------|---|
| **Web Search** | "Augment LLM responses with real-time web search results" | Web Search guide |
| **PDF Inputs** | "Parse and extract content from uploaded PDF files" | PDF Inputs guide |
| **Response Healing** | "Automatically fix malformed JSON responses from LLMs" | Response Healing guide |

## API Implementation

Plugins are activated by including a `plugins` array in chat completion requests. Each plugin has an `id` and optional configuration parameters.

### Example (TypeScript):
```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer {{API_KEY_REF}}',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: '{{MODEL}}',
    messages: [{ role: 'user', content: 'What are the latest developments in AI?' }],
    plugins: [{ id: 'web' }]
  }),
});
```

## Multiple Plugins

Combine plugins in a single request with custom parameters:

```json
{
  "plugins": [
    { "id": "web", "max_results": 3 },
    { "id": "response-healing" }
  ]
}
```

## Default Configuration

Admins and users can set organization-wide plugin defaults via Settings > Plugins. The "Prevent overrides" option enforces settings across all requests.

### Precedence Order:
1. Request-level plugin settings
2. Account default settings

## Disabling Default Plugins

Override default enabled plugins for specific requests:

```json
{
  "plugins": [{ "id": "web", "enabled": false }]
}
```

## Model Variant Shortcuts

Append `:online` to model IDs as shorthand for web search:

```json
{ "model": "openai/gpt-5.2:online" }
```

Equals: `"model": "openai/gpt-5.2"` with `"plugins": [{ "id": "web" }]`
