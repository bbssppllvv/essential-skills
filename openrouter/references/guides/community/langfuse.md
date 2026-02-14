# Langfuse Integration with OpenRouter

## Overview

"Langfuse provides observability and analytics for LLM applications." Since OpenRouter uses the OpenAI API schema, you can leverage Langfuse's native OpenAI SDK integration to automatically trace OpenRouter calls.

## Setup Instructions

### Installation
```bash
pip install langfuse openai
```

### Environment Configuration
Configure your credentials for both services:

```python
import os

# Langfuse API keys
LANGFUSE_SECRET_KEY="sk-lf-..."
LANGFUSE_PUBLIC_KEY="pk-lf-..."
# EU: https://cloud.langfuse.com
# US: https://us.cloud.langfuse.com

# OpenRouter API key
os.environ["OPENAI_API_KEY"] = "${API_KEY_REF}"
```

## Basic Implementation

Create an OpenAI client pointing to OpenRouter's endpoint:

```python
from langfuse.openai import openai

client = openai.OpenAI(
    base_url="https://openrouter.ai/api/v1",
    default_headers={
        "HTTP-Referer": "<YOUR_SITE_URL>",
        "X-Title": "<YOUR_SITE_NAME>",
    }
)

response = client.chat.completions.create(
    model="anthropic/claude-3.5-sonnet",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me a fun fact about space."}
    ],
    name="fun-fact-request"
)

print(response.choices[0].message.content)
```

## Advanced Tracing with Decorators

The `@observe()` decorator captures nested function execution:

```python
from langfuse import observe
from langfuse.openai import openai

client = openai.OpenAI(
    base_url="https://openrouter.ai/api/v1",
)

@observe()
def analyze_text(text: str):
    summary = summarize_text(text).choices[0].message.content
    sentiment = analyze_sentiment(summary).choices[0].message.content
    return {"summary": summary, "sentiment": sentiment}

@observe()
def summarize_text(text: str):
    return client.chat.completions.create(
        model="openai/gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "You summarize texts concisely."},
            {"role": "user", "content": f"Summarize:\n{text}"}
        ],
        name="summarize-text"
    )

@observe()
def analyze_sentiment(summary: str):
    return client.chat.completions.create(
        model="openai/gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "You analyze sentiment."},
            {"role": "user", "content": f"Analyze sentiment:\n{summary}"}
        ],
        name="analyze-sentiment"
    )
```

## Additional Resources

- Langfuse OpenRouter Integration documentation
- OpenRouter Quick Start Guide
- Langfuse decorator documentation for Python
