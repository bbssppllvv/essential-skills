# Responses API Beta Reasoning

## Overview

The Responses API Beta provides advanced reasoning capabilities that allow models to demonstrate their internal thinking processes with adjustable computational effort levels.

## Configuration

To enable reasoning, use the `reasoning` parameter in your API request with an `effort` setting. Here's the basic structure:

```json
{
  "model": "openai/o4-mini",
  "input": "Your question here",
  "reasoning": {
    "effort": "high"
  },
  "max_output_tokens": 9000
}
```

## Effort Levels

Four computational intensity options are available:

- **minimal**: Basic reasoning with minimal computational effort
- **low**: Light reasoning for simple problems
- **medium**: Balanced reasoning for moderate complexity
- **high**: Deep reasoning for complex problems

## Key Features

**Response Structure**: When reasoning is enabled, responses include a dedicated reasoning block containing encrypted content and a summary of the thinking process, followed by the assistant's message.

**Token Usage**: The response includes breakdown details showing reasoning tokens separately from output tokens, helping developers understand computational costs.

**Streaming Support**: The API allows real-time streaming of reasoning development through specific event types like `response.reasoning.delta`.

**Multi-turn Conversations**: Reasoning works within conversation contexts, maintaining coherence across multiple message exchanges.

## Recommendations

- Align effort levels to problem complexity
- Account for increased token consumption when budgeting
- Use streaming for improved user experience with lengthy reasoning chains
- Provide adequate context for effective reasoning processes
