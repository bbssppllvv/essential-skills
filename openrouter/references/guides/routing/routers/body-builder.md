# Body Builder Documentation

## Overview

Body Builder is a tool that transforms natural language prompts into structured OpenRouter API requests. It enables parallel execution of the same task across multiple models simultaneously. The service is complimentary--there are no costs for generating request bodies.

## Key Functionality

Body Builder leverages AI to interpret user intent and produce valid OpenRouter API request bodies. Users simply describe their objective and specify which models to use, receiving ready-to-execute JSON requests in return.

## How to Use

Users can access Body Builder through three implementation methods:

**TypeScript SDK:**
```typescript
const completion = await openRouter.chat.send({
  model: 'openrouter/bodybuilder',
  messages: [{ role: 'user', content: 'Count to 10 using Claude Sonnet and GPT-5' }],
});
const generatedRequests = JSON.parse(completion.choices[0].message.content);
```

**Direct API (fetch or Python):**
Both approaches POST to `https://openrouter.ai/api/v1/chat/completions` with model set to `openrouter/bodybuilder`.

## Response Structure

The service returns JSON containing an array of request objects:

```json
{
  "requests": [
    { "model": "anthropic/claude-sonnet-4.5", "messages": [...] },
    { "model": "openai/gpt-5.1", "messages": [...] }
  ]
}
```

## Practical Applications

- **Model Comparison:** Evaluate how different models tackle identical tasks
- **Redundancy:** Obtain responses from multiple providers for validation
- **Testing:** Assess prompts across models to identify optimal matches
- **Research:** Identify which models excel at specific domains

## Supported Models

The system recognizes common model names and aliases:
- "Claude Sonnet" maps to `anthropic/claude-sonnet-4.5`
- "Claude Opus" maps to `anthropic/claude-opus-4.5`
- "GPT-5" maps to `openai/gpt-5.1`
- "Gemini" maps to `google/gemini-3-pro-preview`
- "DeepSeek" maps to `deepseek/deepseek-v3.2`

## Cost Structure

Request generation is free. Standard model pricing applies when executing the generated requests.

## Technical Constraints

- Requires message-format input
- Preserves system messages from user input
- Uses minimal required fields by default
