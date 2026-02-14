# OpenAI SDK Integration with OpenRouter

## Overview

OpenRouter supports integration with the official OpenAI SDK for both Python and TypeScript, allowing developers to use OpenRouter as a drop-in replacement by changing the base URL and API key.

## Installation

- **Python**: `pip install openai`
- **TypeScript/JavaScript**: `npm i openai`

An automated migration tool is also available via Grit for code updates:

```bash
npx @getgrit/launcher openrouter
```

## Implementation Examples

### TypeScript Implementation

```typescript
import OpenAI from "openai"

const openai = new OpenAI({
  baseURL: "https://openrouter.ai/api/v1",
  apiKey: "$OPENROUTER_API_KEY",
  defaultHeaders: {
    "HTTP-Referer": "<YOUR_SITE_URL>",
    "X-Title": "<YOUR_SITE_NAME>",
  },
})

async function main() {
  const completion = await openai.chat.completions.create({
    model: "openai/gpt-4o",
    messages: [
      { role: "user", content: "Say this is a test" }
    ],
  })

  console.log(completion.choices[0].message)
}
main();
```

### Python Implementation

```python
from openai import OpenAI
from os import getenv

client = OpenAI(
  base_url="https://openrouter.ai/api/v1",
  api_key=getenv("OPENROUTER_API_KEY"),
)

completion = client.chat.completions.create(
  model="openai/gpt-4o",
  extra_headers={
    "HTTP-Referer": "<YOUR_SITE_URL>",
    "X-Title": "<YOUR_SITE_NAME>",
  },
  messages=[
    {
      "role": "user",
      "content": "Say this is a test",
    },
  ],
)
print(completion.choices[0].message.content)
```

## Key Features

Both implementations allow developers to leverage OpenRouter's model selection capabilities while maintaining compatibility with standard OpenAI SDK patterns. The key configuration steps involve setting the base URL to `https://openrouter.ai/api/v1` and providing proper authentication credentials.

The Python implementation accepts optional site metadata through extra headers and supports OpenRouter-specific parameters via the `extra_body` parameter for advanced model selection features.
