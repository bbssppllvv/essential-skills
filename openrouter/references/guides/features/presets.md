# Presets - Configuration Management for AI Models

## Overview

"Presets allow you to separate your LLM configuration from your code." You can create and manage these settings through OpenRouter's web application to control routing, model selection, system prompts, and parameters, then reference them in API requests.

## What Presets Do

Named configurations bundle all settings needed for specific use cases. Examples include:

* "email-copywriter" for marketing content generation
* "inbound-classifier" for customer inquiry categorization
* "code-reviewer" for pull request analysis

Each preset manages:
* Provider routing preferences (sorted by price, latency, etc.)
* Model selection with fallback options
* System prompts
* Generation parameters (temperature, top_p, etc.)
* Provider inclusion/exclusion rules

## Getting Started

1. Create a preset via the settings interface, selecting a model and configuring provider routing
2. Make an API request referencing the preset:

```json
{
  "model": "@preset/ravenel-bridge",
  "messages": [
    {
      "role": "user",
      "content": "What's your opinion of the Golden Gate Bridge?"
    }
  ]
}
```

## Key Advantages

**Separation of Concerns**: Keep application code distinct from LLM configuration for cleaner, more maintainable code.

**Rapid Iteration**: Update configurations without code deployments--switch models, adjust prompts, modify parameters, and change provider preferences on demand.

## Three Usage Methods

**1. Direct Reference**
```json
{"model": "@preset/email-copywriter"}
```

**2. Preset Field**
```json
{"model": "openai/gpt-4", "preset": "email-copywriter"}
```

**3. Combined Approach**
```json
{"model": "openai/gpt-4@preset/email-copywriter"}
```

## Important Details

* Organization accounts allow all members to access shared presets
* Version history enables rollback capability; API requests always use the latest version
* Request parameters are shallow-merged with preset configuration
