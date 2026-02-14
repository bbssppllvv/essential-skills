# OpenRouter Python SDK Documentation

## Overview

The OpenRouter Python SDK is a type-safe toolkit for building AI applications with access to 300+ language models through a unified API.

## Key Features

**Auto-generated from API specifications:** The SDK automatically updates with new models and parameters, making them instantly available in IDE autocomplete without manual version management.

**Type-safe by default:** All parameters, response fields, and configuration options include Python type hints and Pydantic validation. Invalid configurations trigger clear, actionable error messages at runtime.

**Streaming support:** Type-safe streaming with full type information for each event in the stream.

**Async support:** Built-in async methods via `send_async()` for non-blocking operations.

## Installation

```bash
# Using uv (recommended)
uv add openrouter

# Using pip
pip install openrouter

# Using poetry
poetry add openrouter
```

Requires Python 3.9+. Get your API key from [openrouter.ai/settings/keys](https://openrouter.ai/settings/keys).

## Basic Usage

```python
from openrouter import OpenRouter
import os

with OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY")
) as client:
    response = client.chat.send(
        model="minimax/minimax-m2",
        messages=[
            {"role": "user", "content": "Hello!"}
        ]
    )
    print(response.choices[0].message.content)
```

## Streaming

```python
stream = client.chat.send(
    model="minimax/minimax-m2",
    messages=[{"role": "user", "content": "Write a story"}],
    stream=True
)

for event in stream:
    content = event.choices[0].delta.content if event.choices else None
```

## Async Operations

```python
import asyncio
from openrouter import OpenRouter
import os

async def main():
    async with OpenRouter(
        api_key=os.getenv("OPENROUTER_API_KEY")
    ) as client:
        response = await client.chat.send_async(
            model="minimax/minimax-m2",
            messages=[{"role": "user", "content": "Hello"}]
        )
        print(response.choices[0].message.content)

asyncio.run(main())
```

## Status

The Python SDK and documentation are currently in beta. Issues should be reported on [GitHub](https://github.com/OpenRouterTeam/python-sdk/issues).
