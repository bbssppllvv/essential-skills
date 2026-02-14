# Thinking Variant

> Enable extended reasoning with :thinking

The `:thinking` variant enables extended reasoning capabilities for complex problem-solving tasks.

## Usage

Append `:thinking` to any model ID:

```json
{
  "model": "deepseek/deepseek-r1:thinking"
}
```

## Details

Thinking variants provide access to models with extended reasoning capabilities, allowing for more thorough analysis and step-by-step problem solving. This is particularly useful for complex tasks that benefit from chain-of-thought reasoning.

## Reasoning Tokens

For models that support it, the OpenRouter API can return Reasoning Tokens, also known as thinking tokens. OpenRouter normalizes the different ways of customizing the amount of reasoning tokens that the model will use, providing a unified interface across different providers.

To activate reasoning tokens, include `"include_reasoning": true` in your API request. When enabled, reasoning output appears in the `"reasoning"` field of each message response.

You can also enable reasoning for any thinking model on OpenRouter using:

```json
{
  "reasoning": {
    "enabled": true
  }
}
```

Reasoning tokens provide transparent visibility into model decision-making steps and work across both the web interface and programmatic API access.

### Supported Models

Reasoning tokens are initially available for:

- **DeepSeek R1** models and derived variants
- **Gemini Thinking** models
- Other models with thinking variant support (e.g., GLM-4.5-Air which supports hybrid inference modes, offering a "thinking mode" for advanced reasoning and tool use, and a "non-thinking mode" for real-time interaction)

## See Also

- [Reasoning Tokens](/docs/best-practices/reasoning-tokens)
