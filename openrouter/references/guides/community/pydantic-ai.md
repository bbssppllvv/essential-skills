# PydanticAI Integration with OpenRouter

## Overview

"PydanticAI provides a high-level interface for working with various LLM providers, including OpenRouter."

## Setup Instructions

### Installation

Install the required package using pip:

```bash
pip install 'pydantic-ai-slim[openai]'
```

### Configuration Steps

To integrate OpenRouter with PydanticAI, leverage its OpenAI-compatible interface. Here's the implementation:

```python
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel

model = OpenAIModel(
    "anthropic/claude-3.5-sonnet",  # or any other OpenRouter model
    base_url="https://openrouter.ai/api/v1",
    api_key="sk-or-...",
)

agent = Agent(model)
result = await agent.run("What is the meaning of life?")
print(result)
```

## Key Configuration Parameters

- **Model identifier**: Specify any supported OpenRouter model (example: `anthropic/claude-3.5-sonnet`)
- **Base URL**: `https://openrouter.ai/api/v1`
- **Authentication**: Provide your OpenRouter API key in the `sk-or-` format

For additional configuration options and usage patterns, consult the [PydanticAI documentation](https://ai.pydantic.dev/models/#api_key-argument).
