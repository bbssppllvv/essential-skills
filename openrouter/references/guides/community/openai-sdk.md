# OpenAI SDK Integration with OpenRouter

## Overview

OpenRouter supports integration with the official OpenAI SDK for both Python and TypeScript implementations.

## Installation

- **Python**: `pip install openai`
- **TypeScript/JavaScript**: `npm i openai`

**Migration Tip**: The page references Grit as an automated code migration tool, allowing developers to run `npx @getgrit/launcher openrouter` for automatic conversion of existing codebases.

## Implementation Examples

### TypeScript Configuration

The TypeScript example demonstrates:
- Initializing OpenAI client with OpenRouter's base URL (`https://openrouter.ai/api/v1`)
- Setting API key from environment variables
- Including optional default headers
- Creating chat completions with specified models
- Handling async responses

### Python Configuration

The Python implementation shows:
- Client initialization pointing to OpenRouter's API endpoint
- API key retrieval from the `OPENROUTER_API_KEY` environment variable
- Optional HTTP headers for site tracking (referrer and title)
- Comment-blocked configuration for accessing OpenRouter-specific parameters
- Message formatting and response extraction

## Key Features

Both implementations allow developers to leverage OpenRouter's model selection capabilities while maintaining compatibility with standard OpenAI SDK patterns.
