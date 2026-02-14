# Message Formats - OpenRouter SDK Documentation

## Overview

The OpenRouter SDK provides conversion utilities between message formats, enabling seamless migration from other APIs. Users can work with OpenAI chat format, Anthropic Claude format, and OpenResponses format interchangeably.

## OpenAI Chat Format

### fromChatMessages()

This function transforms OpenAI-style message arrays into OpenResponses input. It supports standard roles including system, user, and assistant messages.

**Supported Roles:**
- `system` - System instructions
- `user` - User messages
- `assistant` - Assistant responses
- `developer` - Developer instructions
- `tool` - Tool response messages

### toChatMessage()

Converts OpenResponses responses back to chat message objects with role and content properties.

### Tool Message Handling

Tool responses reference previous assistant tool calls via `tool_call_id`, enabling function calling workflows.

## Anthropic Claude Format

### fromClaudeMessages()

Converts Claude-style message arrays to OpenResponses input format.

### toClaudeMessage()

Transforms OpenResponses responses into Claude-compatible message format.

### Content Blocks & Features

Claude format supports:
- **Text and image content blocks** - Mixed content in single messages
- **Tool use blocks** - Named tool invocations with structured input
- **Base64 images** - Both URL and base64-encoded image support
- **Tool result blocks** - Responses to tool calls

### Conversion Limitations

Some Claude-specific attributes (like the `is_error` flag on tool results) aren't preserved during conversion, as they're not universally supported across OpenRouter's platform.

## Migration Patterns

### From OpenAI SDK
Replace `openai.chat.completions.create()` with `openrouter.callModel()` using `fromChatMessages()` to wrap existing message arrays.

### From Anthropic SDK
Replace `anthropic.messages.create()` with `openrouter.callModel()` using `fromClaudeMessages()` and adjust parameter names like `max_tokens` to `maxOutputTokens`.

## Multi-Turn Conversations

Accumulate messages across sequential API calls by:
1. Converting responses back to message format using helper functions
2. Appending new user input
3. Re-passing the complete history to subsequent calls

This enables stateful conversations while maintaining format compatibility.
