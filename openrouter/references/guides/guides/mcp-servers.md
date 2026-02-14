# Using MCP Servers with OpenRouter

## Overview

"MCP servers are a popular way of providing LLMs with tool calling abilities, and are an alternative to using OpenAI-compatible tool calling." The integration works by converting Anthropic's MCP tool definitions into OpenAI-compatible formats for use with OpenRouter.

## Setup Requirements

Before implementing this integration, you'll need:
- Python packages installed via pip
- A `.env` file containing your `OPENAI_API_KEY`
- The example assumes a `/Applications` directory exists

## Key Components

### Model Configuration

The example uses `anthropic/claude-3-7-sonnet` as the model through OpenRouter's API endpoint at `https://openrouter.ai/api/v1`.

### Tool Format Conversion

A helper function transforms MCP tool definitions into OpenAI-compatible format:

```python
def convert_tool_format(tool):
    converted_tool = {
        "type": "function",
        "function": {
            "name": tool.name,
            "description": tool.description,
            "parameters": {
                "type": "object",
                "properties": tool.inputSchema["properties"],
                "required": tool.inputSchema["required"]
            }
        }
    }
    return converted_tool
```

### MCP Client Implementation

The documentation provides approximately 100 lines of code implementing an `MCPClient` class that:
- Manages session connections to MCP servers
- Lists available tools from servers
- Processes user queries through the LLM
- Executes tool calls and handles results
- Supports interactive chat loops

## Important Note

"Interacting with MCP servers is more complex than calling a REST endpoint. The MCP protocol is stateful and requires session management."

## Example Usage

The provided implementation demonstrates querying whether Microsoft Office is installed by leveraging filesystem tools and multiple API calls with tool execution and result handling.
