# Auto Router Documentation

## Overview

The Auto Router (`openrouter/auto`) leverages NotDiamond's technology to intelligently select optimal models for your prompts, analyzing complexity and task requirements rather than requiring manual model selection.

## Basic Usage

To implement Auto Router, set the model parameter to `openrouter/auto`. The system analyzes your request and forwards it to the most suitable model from a curated selection.

**Key Implementation Points:**
- Available in TypeScript SDK, fetch API, and Python
- The response includes a `model` field identifying which model was selected
- Standard OpenRouter API endpoint: `https://openrouter.ai/api/v1/chat/completions`

## How the System Works

The routing process follows these steps:

1. **Prompt Analysis** - NotDiamond's system evaluates your input
2. **Model Selection** - The optimal model is determined based on task requirements
3. **Request Forwarding** - Your request goes to the selected model
4. **Response Metadata** - Results include details about which model processed it

## Available Model Pool

The router selects from high-quality options including Claude Sonnet 4.5, Claude Opus 4.5, GPT-5.1, Gemini 3 Pro, DeepSeek 3.2, and other top-performing models. The available pool updates as new versions are released.

## Restricting Model Selection

You can control which models the router considers using the `plugins` parameter with wildcard patterns:

| Pattern | Matches |
|---------|---------|
| `anthropic/*` | All Anthropic models |
| `openai/gpt-5*` | GPT-5 variants |
| `openai/gpt-5.1` | Exact match |
| `*/claude-*` | Any provider with "claude" |

Configuration can happen per-request via API or as defaults through the Settings UI.

## Pricing & Limitations

There is no additional charge beyond standard model rates. The feature requires `messages` format (not `prompt`), supports streaming, and works with all standard OpenRouter features including tool calling.

## Ideal Applications

- General-purpose services with diverse prompt types
- Cost optimization through efficient model selection
- Quality assurance for complex requests
- Testing to identify optimal models for specific use cases
