# OpenRouter Python SDK Documentation

## Overview

The OpenRouter Python SDK is described as "a type-safe toolkit for building AI applications with access to 300+ language models through a unified API."

## Key Features

### Auto-Generated from API Specifications
The SDK automatically generates from OpenRouter's OpenAPI specs and updates with each API change. Models and parameters become available in IDE autocomplete immediately upon release.

### Type Safety
All parameters, response fields, and configuration options include Python type hints and Pydantic validation. Invalid configurations trigger clear, actionable error messages at runtime.

### Streaming Support
The SDK provides type-safe streaming responses with full type information for each event in the stream.

### Async Support
Developers can use `send_async()` for asynchronous API calls within async contexts.

## Installation

**Package Managers:**
- UV (recommended): `uv add openrouter`
- Pip: `pip install openrouter`
- Poetry: `poetry add openrouter`

**Requirements:** Python 3.9 or higher

## Quick Start Example

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

## API Key Setup

Retrieve your API key from openrouter.ai/settings/keys.

## Status

The Python SDK and documentation are currently in beta. Issues should be reported on GitHub.
