# API Reference - OpenRouter SDK

Complete reference documentation for the OpenRouter TypeScript SDK, covering the `callModel` API, `ModelResult` class, tool types, and helper functions.

## callModel Function

Creates responses using the OpenResponses API with multiple consumption patterns.

```typescript
function callModel(request: CallModelInput, options?: RequestOptions): ModelResult
```

### Key Parameters

The `CallModelInput` accepts either a `model` string or `models` array (one required). Additional configuration includes:

- **Input & Instructions**: `input` (OpenResponsesInput) and `instructions` (optional system guidance)
- **Tool Configuration**: `tools` array, `toolChoice`, `parallelToolCalls` toggle
- **Sampling Controls**: `temperature` (0-2), `topP`, `topK`
- **Token Management**: `maxOutputTokens`, `promptCacheKey`
- **Provider Routing**: `provider` preferences for fallbacks and ordering
- **Advanced Features**: `reasoning` config, `plugins`, background execution, session tracking

### ProviderPreferences

Control provider selection through:
- `allowFallbacks` (default: true)
- `requireParameters` (support validation)
- `order` and `only`/`ignore` arrays
- Load balancing via `sort` parameter
- Throughput and latency preferences
- Price limits and quantization filtering

## ModelResult Class

Wrapper providing multiple consumption patterns:

| Method | Returns | Purpose |
|--------|---------|---------|
| `getText()` | `Promise<string>` | Text after tool completion |
| `getResponse()` | `Promise<OpenResponsesNonStreamingResponse>` | Full response with usage metrics |
| `getTextStream()` | `AsyncIterableIterator<string>` | Text deltas |
| `getReasoningStream()` | `AsyncIterableIterator<string>` | Reasoning output (reasoning models) |
| `getToolCalls()` | `Promise<ParsedToolCall[]>` | All tool calls from initial response |
| `getToolStream()` | `AsyncIterableIterator<ToolStreamEvent>` | Tool deltas and results |
| `cancel()` | `Promise<void>` | Cancel stream and consumers |

## Tool Types

Create typed tools using the `tool()` function with Zod schema validation:

```typescript
function tool<TInput, TOutput>(config: ToolConfig): Tool
```

**Tool Types:**

1. **ToolWithExecute**: Regular function-based tools with immediate execution
2. **ToolWithGenerator**: Generator tools supporting streaming events via `eventSchema`
3. **ManualTool**: Tools without execute functions (external handling)

## Stop Conditions

Terminate processing based on custom logic:

```typescript
type StopWhen = StopCondition | StopCondition[];
type StopCondition = (context: StopConditionContext) => boolean | Promise<boolean>;
```

**Built-in Helpers:**
- `stepCountIs(n)` -- stop after n steps
- `hasToolCall(name)` -- stop when specific tool called
- `maxTokensUsed(n)` -- token budget limit
- `maxCost(amount)` -- cost-based cutoff
- `finishReasonIs(reason)` -- stop on finish reason

## Format Helpers

Convert between message formats:

- `fromChatMessages()` -- OpenAI format to OpenResponses
- `toChatMessage()` -- OpenResponses to OpenAI format
- `fromClaudeMessages()` -- Anthropic Claude to OpenResponses
- `toClaudeMessage()` -- OpenResponses to Claude format

## Type Utilities

Infer types from tool definitions:

- `InferToolInput<T>` -- extract input schema type
- `InferToolOutput<T>` -- extract output schema type
- `InferToolEvent<T>` -- extract event schema type

## Context Types

- **TurnContext**: Provides tool call info, turn count, and request context
- **NextTurnParamsContext**: Tracks parameters for subsequent turns
