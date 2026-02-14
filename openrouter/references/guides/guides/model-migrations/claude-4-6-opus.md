# Claude 4.6 Opus Migration Guide

## Overview

This guide covers migrating to Claude 4.6 Opus, which introduces significant changes to how the model handles reasoning and response effort levels.

## Key Changes

Claude 4.6 Opus implements two major updates:

1. **Adaptive Thinking** — The model independently decides reasoning depth based on task requirements, superseding the previous token-budget approach
2. **Max Effort Level** — A new `'max'` setting for the verbosity parameter (Claude 4.6 Opus only)

## Adaptive Thinking Explained

### Default Behavior

When reasoning is enabled without specifying token limits, Claude 4.6 Opus automatically uses adaptive thinking. "Claude automatically determines the appropriate amount of reasoning based on task complexity" without requiring developers to preset token budgets.

### Implementation Examples

**Adaptive (recommended):**
```json
{
  "model": "anthropic/claude-4.6-opus",
  "reasoning": { "enabled": true }
}
```

**Budget-based (still functional):**
```json
{
  "model": "anthropic/claude-4.6-opus",
  "reasoning": { "enabled": true, "max_tokens": 10000 }
}
```

## Max Effort Parameter

The new verbosity setting offers enhanced response thoroughness:

```json
{
  "model": "anthropic/claude-4.6-opus",
  "verbosity": "max"
}
```

Note: This feature only works on Claude 4.6 Opus; other models revert to `'high'`.

## Parameter Distinctions

| Parameter | Function | Opus 4.6 Behavior |
|-----------|----------|------------------|
| `verbosity` | Output detail | Supports `'max'` |
| `reasoning.effort` | Thinking budget | Ignored (adaptive used) |

## Backward Compatibility

Existing implementations remain functional. Budget-based thinking activates when `reasoning.max_tokens` is explicitly set. Previous model versions (4.5 Opus, 3.7 Sonnet) maintain their original functionality unchanged.
