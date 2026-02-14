# LangChain Integration with OpenRouter

## Overview

OpenRouter integrates with LangChain by configuring the `base_url` parameter to point to OpenRouter's API endpoint. This enables developers to leverage LangChain's standardized interface for chat models across multiple providers.

## Key Resources

- LangChain Python framework: Available on [GitHub](https://github.com/langchain-ai/langchain)
- Example implementation with Streamlit: [GitHub repository](https://github.com/alexanderatallah/openrouter-streamlit)
- Reference documentation: [LangChain Models documentation](https://docs.langchain.com/oss/python/langchain/models#initialize-a-model)

## Implementation Examples

### TypeScript

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { HumanMessage, SystemMessage } from "@langchain/core/messages";

const chat = new ChatOpenAI(
  {
    model: '<model_name>',
    temperature: 0.8,
    streaming: true,
    apiKey: '$OPENROUTER_API_KEY',
  },
  {
    baseURL: 'https://openrouter.ai/api/v1',
    defaultHeaders: {
      'HTTP-Referer': '<YOUR_SITE_URL>',
      'X-Title': '<YOUR_SITE_NAME>',
    },
  },
);

const response = await chat.invoke([
  new SystemMessage("You are a helpful assistant."),
  new HumanMessage("Hello, how are you?"),
]);
```

### Python (init_chat_model)

```python
from langchain.chat_models import init_chat_model
from os import getenv
from dotenv import load_dotenv

load_dotenv()

model = init_chat_model(
    model="<model_name>",
    model_provider="openai",
    base_url="https://openrouter.ai/api/v1",
    api_key=getenv("OPENROUTER_API_KEY"),
    default_headers={
        "HTTP-Referer": getenv("YOUR_SITE_URL"),
        "X-Title": getenv("YOUR_SITE_NAME"),
    }
)

response = model.invoke("What NFL team won the Super Bowl in the year Justin Bieber was born?")
print(response.content)
```

### Python (ChatOpenAI Direct)

```python
from langchain_openai import ChatOpenAI
from os import getenv
from dotenv import load_dotenv

load_dotenv()

llm = ChatOpenAI(
    api_key=getenv("OPENROUTER_API_KEY"),
    base_url="https://openrouter.ai/api/v1",
    model="<model_name>",
    default_headers={
        "HTTP-Referer": getenv("YOUR_SITE_URL"),
        "X-Title": getenv("YOUR_SITE_NAME"),
    }
)

response = llm.invoke("What NFL team won the Super Bowl in the year Justin Bieber was born?")
print(response.content)
```

## Configuration Notes

Optional headers (`HTTP-Referer` and `X-Title`) support site rankings on the OpenRouter platform. For complete details on LangChain's model interface, consult the [official documentation](https://docs.langchain.com/oss/python/langchain/models#initialize-a-model).
