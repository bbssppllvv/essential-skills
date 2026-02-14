# Stop Conditions - OpenRouter SDK Documentation

## Overview

The `stopWhen` parameter enables control over multi-turn execution by defining conditions that halt processing. Users can leverage built-in helpers or implement custom logic to stop based on step count, tool invocations, cost, or token usage.

## Built-in Stop Conditions

### stepCountIs(n)
Halts execution after reaching a specified number of steps:
```typescript
stopWhen: stepCountIs(10)
```

### hasToolCall(name)
Terminates when a particular tool gets invoked:
```typescript
stopWhen: hasToolCall('finish')
```

### maxTokensUsed(n)
Stops after consuming a set token threshold:
```typescript
stopWhen: maxTokensUsed(5000)
```

### maxCost(amount)
Halts upon reaching a monetary spending limit:
```typescript
stopWhen: maxCost(1.00)
```

### finishReasonIs(reason)
Terminates based on model completion reason:
```typescript
stopWhen: finishReasonIs('stop')
```

## Combining Multiple Conditions

Pass an array to activate an OR logic pattern -- execution stops when any condition triggers:
```typescript
stopWhen: [
  stepCountIs(10),
  maxCost(0.50),
  hasToolCall('finish')
]
```

## Custom Conditions

Define bespoke logic by providing a function receiving `StopConditionContext`:

```typescript
stopWhen: ({ steps }) => {
  if (steps.length >= 20) return true;
  const lastStep = steps[steps.length - 1];
  if (lastStep && !lastStep.toolCalls?.length) return true;
  return false;
}
```

### StepResult Properties
Each completed step includes:
- `response` - Model response object
- `toolCalls` - Array of parsed tool invocations
- `toolResults` - Execution results from tools
- `tokens` - Input, output, and cached token counts
- `cost` - Financial cost of the step

## Advanced Patterns

**Time-Based Stopping** - Enforce duration limits by comparing elapsed milliseconds against thresholds.

**Content-Based Stopping** - Inspect response text for keywords or completion markers.

**Quality-Based Stopping** - Exit when tool results meet specified evaluation scores.

## Migration Guidance

The legacy `maxToolRounds` parameter has been superseded by `stopWhen: stepCountIs(n)`.

Default behavior when `stopWhen` is unspecified defaults to `stepCountIs(5)`.

## Best Practices

- Always include hard limits (step count and cost) to prevent unbounded execution
- Log stopping rationales for debugging and monitoring
- Test conditions with conservative thresholds before production deployment
