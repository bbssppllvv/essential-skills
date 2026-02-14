# Message Transforms

## Overview

OpenRouter offers a message transformation feature to handle prompts exceeding a model's context window capacity. This functionality is implemented through a custom parameter called `transforms`.

## Basic Implementation

The feature can be activated using the following configuration:

```typescript
{
  transforms: ["middle-out"],
  messages: [...],
  model // Works with any model
}
```

## How It Works

The middle-out compression strategy operates by removing or shortening content from the center of your prompt until it fits within the model's context limits. As noted in the documentation, this approach proves useful "for situations where perfect recall is not required."

### Message Count Limitations

Beyond token constraints, some models enforce maximum message limits. The documentation notes that "Anthropic's Claude models enforce a maximum of {anthropicMaxMessagesCount} messages." When this threshold is exceeded with middle-out enabled, the system preserves half the messages from the conversation's beginning and half from its end.

### Model Selection Process

When middle-out compression activates, OpenRouter prioritizes models whose context length reaches at least 50% of your required tokens (input plus completion). If no models meet this threshold, the system defaults to whichever model offers the largest available context window.

## Default Behavior

Models with 8,192 tokens or less context automatically apply middle-out compression by default. You can disable this by setting `transforms: []` in your request.

## Technical Rationale

The compression targets the middle section because "LLMs pay less attention to the middle of sequences," making this area ideal for content reduction.
