# Next Turn Params Documentation

## Overview

`nextTurnParams` enables tools to modify model parameters for subsequent conversation turns, supporting skills systems, progressive context building, and adaptive behavior.

## Core Concept

Rather than just returning results, tools can reshape how the model responds next. The feature addresses:

- **Skills/Plugins**: Activate domain-specific instructions when needed
- **Progressive Context**: Accumulate information across tool invocations
- **Adaptive Behavior**: Adjust model settings based on tool outcomes
- **Clean Separation**: Tools control their own contextual requirements

## Execution Flow

The processing sequence follows this pattern:

1. Model generates tool calls
2. All tool `execute` functions run
3. `nextTurnParams` functions process (in tools array order)
4. Modified parameters apply to next model turn
5. Cycle repeats until model stops calling tools

## Available Context Properties

Functions receive validated input parameters and context containing:

| Property | Type | Purpose |
|----------|------|---------|
| `input` | OpenResponsesInput | Message history |
| `model` | string \| undefined | Active model |
| `models` | string[] \| undefined | Fallback models |
| `instructions` | string \| undefined | System instructions |
| `temperature` | number \| undefined | Generation randomness |
| `maxOutputTokens` | number \| undefined | Output limit |
| `topP` | number \| undefined | Nucleus sampling |
| `topK` | number \| undefined | Token sampling |

## Modifiable Parameters

Tools can adjust any `CallModelInput` setting: message history (`input`), model selection, system instructions, temperature, and token limits.

## Practical Patterns

**Research Accumulation**: Build context as investigation proceeds by appending findings to instructions.

**Model Upgrades**: Switch to more capable models when complexity analysis reveals challenging tasks.

**Multi-Skill Loading**: Load multiple skill definitions simultaneously by injecting formatted content into message history.

**Language Adaptation**: Dynamically modify instructions based on user language preferences.

## Key Best Practices

- **Idempotency**: Check if context already exists before adding duplicates
- **Type Safety**: Use proper fallbacks when accessing optional context properties
- **Minimal Changes**: Only modify necessary parameters to avoid unintended side effects
