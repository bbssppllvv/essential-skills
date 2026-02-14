# Working with Items - OpenRouter SDK Documentation

## Overview

The OpenRouter SDK's `callModel` implements an items-based streaming model rather than traditional message-based streaming. Items are emitted multiple times with the same ID but progressively updated content.

## Key Differences from Traditional Streaming

| Aspect | Traditional | Items-Based |
|--------|-----------|-----------|
| Approach | Accumulate text chunks | Replace items by ID |
| Message types | Single type | Multiple types |
| Content reconstruction | Manual, at completion | Complete each emission |
| State management | Manual accumulation | Natural React integration |

## Supported Item Types

The `getItemsStream()` method yields these item categories:

- `message` - Assistant text responses
- `function_call` - Tool invocations with arguments
- `reasoning` - Model thinking processes
- `web_search_call` - Web search operations
- `file_search_call` - File search operations
- `image_generation_call` - Image generation operations
- `function_call_output` - Tool execution results

## Streaming Pattern Example

Items maintain consistent IDs across iterations while content updates:

```typescript
// First emission
{ id: "msg_123", type: "message", content: [{ type: "output_text", text: "Hello" }] }

// Second emission (same ID, updated content)
{ id: "msg_123", type: "message", content: [{ type: "output_text", text: "Hello world" }] }
```

## React Implementation

Using a Map structure simplifies state management:

```tsx
const [items, setItems] = useState<Map<string, StreamableOutputItem>>(new Map());

for await (const item of result.getItemsStream()) {
  setItems((prev) => new Map(prev).set(item.id, item));
}
```

## Migration Path

The deprecated `getNewMessagesStream()` is replaced by `getItemsStream()`, which includes additional item types beyond messages alone.
