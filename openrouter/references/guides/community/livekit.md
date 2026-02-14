# LiveKit Integration with OpenRouter

## Overview

LiveKit Agents is "an open-source framework for building voice AI agents." The OpenRouter integration provides unified API access to 300+ models with automatic fallback capabilities.

## Setup Requirements

**Installation:**
```bash
uv add "livekit-agents[openai]~=1.2"
```

Users must obtain an OpenRouter API key and configure it as `OPENROUTER_API_KEY` in their environment file.

## Core Implementation

The basic implementation uses the `with_openrouter` method to instantiate an LLM:

```python
from livekit.plugins import openai

session = AgentSession(
    llm=openai.LLM.with_openrouter(model="anthropic/claude-sonnet-4.5"),
)
```

## Configuration Options

**Fallback Models:** Specify alternative models for resilience:
```python
llm = openai.LLM.with_openrouter(
    model="openai/gpt-4o",
    fallback_models=["anthropic/claude-sonnet-4", "openai/gpt-5-mini"],
)
```

**Provider Routing:** Control inference provider selection with ordering, fallback permissions, and latency-based sorting.

**Web Search:** Activate search functionality through the OpenRouterWebPlugin with configurable result limits.

**Analytics:** Include site URL and application name for tracking purposes.

## Additional Resources

Documentation links include the official LiveKit OpenRouter plugin guide, GitHub repository, and model catalog.
