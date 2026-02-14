# Structured Outputs Documentation

## Overview

OpenRouter enables structured outputs for compatible models, allowing you to enforce JSON Schema validation on responses. This capability ensures "consistent, well-formatted responses that can be reliably parsed by your application."

### Key Benefits

The feature provides several advantages:
- Enforce specific JSON Schema validation
- Obtain consistent, type-safe outputs
- Prevent parsing errors and hallucinated fields
- Streamline response handling

## Implementation

To enable structured outputs, include a `response_format` parameter with `type: json_schema`:

```json
{
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "weather",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "location": { "type": "string" },
          "temperature": { "type": "number" },
          "conditions": { "type": "string" }
        },
        "required": ["location", "temperature", "conditions"],
        "additionalProperties": false
      }
    }
  }
}
```

## Supported Models

Structured outputs work with:
- OpenAI models (GPT-4o and later)
- Google Gemini models
- Anthropic models (Sonnet 4.5, Opus 4.1)
- Most open-source models
- All Fireworks-provided models

## Best Practices

1. Include clear descriptions in schema properties
2. Always set `strict: true` to enforce exact schema compliance

## Additional Features

The documentation mentions streaming support and a Response Healing plugin to mitigate JSON formatting issues in non-streaming requests.
