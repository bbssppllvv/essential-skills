# Response Healing Documentation

## Overview

The Response Healing plugin automatically validates and repairs malformed JSON responses from AI models. It activates for non-streaming requests when you use `response_format` with either `type: "json_schema"` or `type: "json_object"`.

## Key Features

The plugin provides:

* **Automatic JSON repair**: Corrects missing brackets, commas, quotes, and syntax errors
* **Markdown extraction**: Pulls JSON from markdown code blocks

## Common Issues Fixed

The plugin handles several frequent problems in LLM outputs:

**JSON Syntax Errors** -- Missing closing brackets or other structural issues

**Markdown Code Blocks** -- Extracts JSON wrapped in triple backticks

**Mixed Text and JSON** -- Isolates JSON when accompanied by surrounding text

**Trailing Commas** -- Removes invalid trailing commas in objects

**Unquoted Keys** -- Converts JavaScript-style object notation to valid JSON

## Implementation

To use Response Healing, include it in your API request:

```json
{
  "response_format": {
    "type": "json_schema",
    "json_schema": { /* your schema */ }
  },
  "plugins": [
    { "id": "response-healing" }
  ]
}
```

## Important Limitations

The documentation notes that "Response Healing only applies to non-streaming requests." Additionally, some heavily malformed or truncated JSON may remain unrepairable, particularly when `max_tokens` cuts off the response mid-JSON.
