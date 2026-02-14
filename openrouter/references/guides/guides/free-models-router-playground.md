# Free Models Router Documentation

## Overview

OpenRouter provides cost-free AI inference through its Free Models Router feature. Users can access this via the Chat Playground at openrouter.ai/chat without spending money on API calls.

## Core Functionality

The Free Models Router uses the identifier `openrouter/free` and automatically selects appropriate free models based on request requirements. The system intelligently filters options for specific capabilities like image recognition, function calling, and structured outputs.

## Getting Started in Chat Playground

**Step 1:** Navigate to the Chat Playground interface

**Step 2:** Use the Add Model button (keyboard: Cmd+K or Ctrl+K) and search for "free" to view available options

**Step 3:** Select "Free Models Router" from the results

**Step 4:** Begin chatting; the system displays which model handled each response

## Individual Model Selection

Users preferring specific models can choose from options labeled "(free)" such as Trinity Large Preview, Trinity Mini, DeepSeek R1, and various Llama variants.

## API Implementation

The router integrates with programmatic requests by specifying the model parameter:

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openrouter/free",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Important Constraints

"Free models may have different rate limits and availability compared to paid models." These options suit experimentation and modest usage volumes rather than production-grade reliability needs.
