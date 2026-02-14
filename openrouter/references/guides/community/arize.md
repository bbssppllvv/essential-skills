# Arize Integration with OpenRouter

## Overview

"Arize provides observability and tracing for LLM applications." Since OpenRouter maintains OpenAI API compatibility, developers can leverage Arize's OpenInference auto-instrumentation to monitor their OpenRouter requests automatically.

## Setup Requirements

Install the necessary packages:
```bash
pip install openinference-instrumentation-openai openai arize-otel
```

You'll need an OpenRouter API key and Arize credentials (Space ID and API Key).

## Why This Works

OpenRouter functions seamlessly with Arize because it presents "a fully OpenAI-API-compatible endpoint" at `/v1`, allowing existing OpenAI SDKs to work without modification by simply redirecting the base URL.

## Implementation

Configure your OpenAI client to point toward OpenRouter:

```python
from arize.otel import register
from openinference.instrumentation.openai import OpenAIInstrumentor
import openai

tracer_provider = register(
    space_id="your-space-id",
    api_key="your-arize-api-key",
    project_name="your-project-name",
)

OpenAIInstrumentor().instrument(tracer_provider=tracer_provider)

client = openai.OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="your_openrouter_api_key",
)
```

## What Gets Monitored

The instrumentation captures request/response data, model information, token usage, timing metrics, and error details.

## Troubleshooting

Ensure you're using your OpenRouter API key (not OpenAI's) and consult OpenRouter's model list for exact naming conventions.
