# Tool & Function Calling with OpenRouter

## Overview

Tool calling enables LLMs to request external tools without directly executing them. The model suggests which tool to use, you execute it separately, and then return results to the LLM for synthesis into an answer.

OpenRouter standardizes tool calling across different models and providers. You can find compatible models by filtering for the `tools` parameter on their models page.

## Core Workflow

The tool calling process involves three sequential steps:

**Step 1: Send Initial Request with Tools**
Submit a message with available tool definitions so the model knows what's accessible.

**Step 2: Execute Tools Locally**
When the model returns `tool_calls`, execute the requested function on your end and prepare results.

**Step 3: Return Results and Get Answer**
Add the tool results back to the conversation and make another API call for the final response.

## Request Structure

Tools follow a standard format defining the function name, description, and parameter schema:

```json
{
  "type": "function",
  "function": {
    "name": "function_name",
    "description": "What this tool does",
    "parameters": {
      "type": "object",
      "properties": { /* parameter definitions */ },
      "required": ["parameter_names"]
    }
  }
}
```

## Practical Example

A complete workflow searches Project Gutenberg for James Joyce books:

1. Define a `searchGutenbergBooks` function that queries an external API
2. Include its specification in the tools parameter
3. The model requests the tool with arguments like `["James", "Joyce"]`
4. Execute the function and return book results
5. Send results back; the model synthesizes them into readable output

The implementation details vary by SDK (TypeScript, Python, or fetch), but the core pattern remains consistent.

## Advanced Capabilities

**Interleaved Thinking:** Models can reason between tool calls, enabling nuanced decision-making with multiple sequential tools. This increases token usage and latency.

**Agentic Loops:** Implement autonomous iteration where the system continuously calls the LLM, processes tool responses, and repeats until receiving a final answer (with iteration limits for safety).

**Streaming:** Handle tool calls within streamed responses by tracking delta updates and detecting finish reasons.

## Configuration Options

- `tool_choice`: Control whether the model uses tools (`"auto"`, `"none"`, or force specific tools)
- `parallel_tool_calls`: Set to `false` to require sequential tool execution instead of simultaneous calls

## Best Practices

Use clear, descriptive function names and comprehensive descriptions explaining when and how to use each tool. Structure parameters with explicit types, examples, and required field designations to help models make precise calls.

Design complementary tools that naturally chain together (search -> details -> inventory) for intuitive multi-step workflows.
