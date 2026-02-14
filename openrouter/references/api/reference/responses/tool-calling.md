# Tool Calling Documentation - OpenRouter Responses API Beta

## Overview

OpenRouter's Responses API Beta provides comprehensive function calling capabilities, enabling models to invoke functions, execute tools concurrently, and manage intricate multi-step workflows.

## Core Concepts

### Tool Definition Structure

Tools follow the OpenAI function calling format with these components:
- **type**: Specified as 'function'
- **name**: Unique identifier for the tool
- **description**: Explains what the function does
- **parameters**: JSON Schema defining required inputs
- **strict**: Optional validation mode

### Tool Choice Control

The API offers three control modes:

| Mode | Behavior |
|------|----------|
| `auto` | Model autonomously determines tool usage |
| `none` | Model abstains from tool invocation |
| `{type: 'function', name: 'X'}` | Enforces specific tool execution |

## Implementation Patterns

### Basic Single Tool

Define a weather function to fetch current conditions at specified locations. The request includes the tool definition, user query, and tool choice setting.

### Multiple Tools

Combine several tools for complex scenarios. The example demonstrates using both weather and calculator tools, allowing the model to select appropriate functions based on user intent.

### Parallel Execution

The API supports simultaneous tool calls. When a user requests multiple independent operations, the model can invoke all relevant tools in a single response cycle, improving efficiency.

## Response Structure

Tool invocations return:
```
Function call objects containing:
- Unique call identifiers
- Function name
- Stringified JSON arguments
- Call status and metadata
```

## Conversation Integration

Include tool outputs in follow-up requests by adding:
- Original `function_call` objects
- `function_call_output` responses
- Assistant message with analysis

The `id` field becomes required for output objects when building conversation history.

## Real-Time Monitoring

Streaming support enables tracking tool execution as it occurs. Monitor events for:
- Tool addition announcements
- Arguments completion signals
- Response generation progress

## Quality Guidelines

1. Create explicit function descriptions with clear use cases
2. Implement accurate JSON schemas matching parameter requirements
3. Anticipate scenarios where tools might not be selected
4. Structure tools for independent parallel operation
5. Preserve tool responses in conversation context for coherent follow-ups
