# Online Variant: Real-Time Web Search

## Overview

The `:online` variant provides real-time web search functionality for models available on OpenRouter.

## Implementation

To activate this feature, append `:online` to your model identifier:

```json
{
  "model": "openai/gpt-5.2:online"
}
```

This approach is functionally identical to invoking the web plugin directly:

```json
{
  "model": "openrouter/auto",
  "plugins": {
    "web": {}
  }
}
```

## Functionality

The Online variant integrates current web search results into model outputs, enabling access to contemporary information and recent developments. This proves especially valuable for questions demanding up-to-date knowledge beyond what models learned during their training phase.

For additional information, consult the [Web Search](/docs/guides/features/plugins/web-search) documentation.
