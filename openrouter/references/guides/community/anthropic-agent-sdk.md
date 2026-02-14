# Anthropic Agent SDK Integration with OpenRouter

## Overview

The Anthropic Agent SDK enables programmatic AI agent development using Python or TypeScript. It integrates with OpenRouter through environment variable configuration.

## Setup Instructions

Configure these environment variables before running your agent:

```bash
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"
export ANTHROPIC_API_KEY="" # Important: Must be explicitly empty
```

## TypeScript Implementation

Install the package:
```bash
npm install @anthropic-ai/claude-agent-sdk
```

Example agent code:
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

async function main() {
  for await (const message of query({
    prompt: "Find and fix the bug in auth.py",
    options: {
      allowedTools: ["Read", "Edit", "Bash"],
    },
  })) {
    if (message.type === "assistant") {
      console.log(message.message.content);
    }
  }
}

main();
```

## Python Implementation

Install the package:
```bash
pip install claude-agent-sdk
```

Example agent code:
```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Find and fix the bug in auth.py",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit", "Bash"]
        )
    ):
        print(message)

asyncio.run(main())
```

## Model Configuration

The SDK supports model override capabilities using environment variables like `ANTHROPIC_DEFAULT_SONNET_MODEL` and `ANTHROPIC_DEFAULT_OPUS_MODEL` to direct agents toward different models available on OpenRouter.
