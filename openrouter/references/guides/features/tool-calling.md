# Tool & Function Calling

## Overview

Tool calling enables LLMs to suggest external tools without directly invoking them. The process follows three steps: the LLM suggests a tool, you execute it locally, and return results for the model to synthesize into a response.

OpenRouter standardizes this interface across models. You can identify compatible models by filtering at [openrouter.ai/models](https://openrouter.ai/models) with the `supported_parameters=tools` query.

## Core Process

**Step 1: Request with Tool Definitions**

The initial request includes tool specifications in JSON format, defining function name, description, and parameters schema.

**Step 2: Local Execution**

After receiving the model's response containing `tool_calls`, you execute the requested function locally using the provided arguments.

**Step 3: Follow-up Request**

Send a second request including the tool results, allowing the model to formulate its final answer based on actual data.

## Implementation Example

### TypeScript SDK Setup

```typescript
import { OpenRouter } from '@openrouter/sdk';

const OPENROUTER_API_KEY = "{{API_KEY_REF}}";
const MODEL = "{{MODEL}}";

const openRouter = new OpenRouter({
  apiKey: OPENROUTER_API_KEY,
});

const task = "What are the titles of some James Joyce books?";

const messages = [
  {
    role: "system",
    content: "You are a helpful assistant."
  },
  {
    role: "user",
    content: task,
  }
];
```

### Python Client Setup

```python
import json, requests
from openai import OpenAI

OPENROUTER_API_KEY = f"{{API_KEY_REF}}"
MODEL = "{{MODEL}}"

openai_client = OpenAI(
  base_url="https://openrouter.ai/api/v1",
  api_key=OPENROUTER_API_KEY,
)

task = "What are the titles of some James Joyce books?"

messages = [
  {
    "role": "system",
    "content": "You are a helpful assistant."
  },
  {
    "role": "user",
    "content": task,
  }
]
```

### Fetch API Setup

```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer {{API_KEY_REF}}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: '{{MODEL}}',
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' },
      {
        role: 'user',
        content: 'What are the titles of some James Joyce books?',
      },
    ],
  }),
});
```

## Tool Definition Example

### TypeScript Implementation

```typescript
async function searchGutenbergBooks(searchTerms: string[]): Promise<Book[]> {
  const searchQuery = searchTerms.join(' ');
  const url = 'https://gutendex.com/books';
  const response = await fetch(`${url}?search=${searchQuery}`);
  const data = await response.json();

  return data.results.map((book: any) => ({
    id: book.id,
    title: book.title,
    authors: book.authors,
  }));
}

const tools = [
  {
    type: 'function',
    function: {
      name: 'searchGutenbergBooks',
      description:
        'Search for books in the Project Gutenberg library based on specified search terms',
      parameters: {
        type: 'object',
        properties: {
          search_terms: {
            type: 'array',
            items: {
              type: 'string',
            },
            description:
              "List of search terms to find books in the Gutenberg library (e.g. ['dickens', 'great'] to search for books by Dickens with 'great' in the title)",
          },
        },
        required: ['search_terms'],
      },
    },
  },
];

const TOOL_MAPPING = {
  searchGutenbergBooks,
};
```

### Python Implementation

```python
def search_gutenberg_books(search_terms):
    search_query = " ".join(search_terms)
    url = "https://gutendex.com/books"
    response = requests.get(url, params={"search": search_query})

    simplified_results = []
    for book in response.json().get("results", []):
        simplified_results.append({
            "id": book.get("id"),
            "title": book.get("title"),
            "authors": book.get("authors")
        })

    return simplified_results

tools = [
  {
    "type": "function",
    "function": {
      "name": "search_gutenberg_books",
      "description": "Search for books in the Project Gutenberg library based on specified search terms",
      "parameters": {
        "type": "object",
        "properties": {
          "search_terms": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "List of search terms to find books in the Gutenberg library (e.g. ['dickens', 'great'] to search for books by Dickens with 'great' in the title)"
          }
        },
        "required": ["search_terms"]
      }
    }
  }
]

TOOL_MAPPING = {
    "search_gutenberg_books": search_gutenberg_books
}
```

## Making Tool Calls

### TypeScript SDK Request

```typescript
const result = await openRouter.chat.send({
  model: '{{MODEL}}',
  tools,
  messages,
  stream: false,
});

const response_1 = result.choices[0].message;
```

### Python Request

```python
request_1 = {
    "model": MODEL,
    "tools": tools,
    "messages": messages
}

response_1 = openai_client.chat.completions.create(**request_1).choices[0].message
```

### Fetch API Request

```typescript
const request_1 = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer {{API_KEY_REF}}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: '{{MODEL}}',
    tools,
    messages,
  }),
});

const data = await request_1.json();
const response_1 = data.choices[0].message;
```

## Processing Tool Calls

### TypeScript Tool Processing

```typescript
messages.push(response_1);

for (const toolCall of response_1.tool_calls) {
  const toolName = toolCall.function.name;
  const { search_params } = JSON.parse(toolCall.function.arguments);
  const toolResponse = await TOOL_MAPPING[toolName](search_params);
  messages.push({
    role: 'tool',
    toolCallId: toolCall.id,
    name: toolName,
    content: JSON.stringify(toolResponse),
  });
}
```

### Python Tool Processing

```python
messages.append(response_1)

for tool_call in response_1.tool_calls:
    tool_name = tool_call.function.name
    tool_args = json.loads(tool_call.function.arguments)
    tool_response = TOOL_MAPPING[tool_name](**tool_args)
    messages.append({
      "role": "tool",
      "tool_call_id": tool_call.id,
      "content": json.dumps(tool_response),
    })
```

## Follow-up Request with Results

### TypeScript

```typescript
const response_2 = await openRouter.chat.send({
  model: '{{MODEL}}',
  messages,
  tools,
  stream: false,
});

console.log(response_2.choices[0].message.content);
```

### Python

```python
request_2 = {
  "model": MODEL,
  "messages": messages,
  "tools": tools
}

response_2 = openai_client.chat.completions.create(**request_2)

print(response_2.choices[0].message.content)
```

### Fetch API

```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer {{API_KEY_REF}}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: '{{MODEL}}',
    messages,
    tools,
  }),
});

const data = await response.json();
console.log(data.choices[0].message.content);
```

## Interleaved Thinking

Models can reason between tool calls, enabling more sophisticated decision-making with multiple chained calls and intermediate analysis. This increases token usage and latency but enables more nuanced workflows.

### Research Request Example

```json
{
  "model": "anthropic/claude-sonnet-4.5",
  "messages": [
    {
      "role": "user",
      "content": "Research the environmental impact of electric vehicles and provide a comprehensive analysis."
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "search_academic_papers",
        "description": "Search for academic papers on a given topic",
        "parameters": {
          "type": "object",
          "properties": {
            "query": {"type": "string"},
            "field": {"type": "string"}
          },
          "required": ["query"]
        }
      }
    },
    {
      "type": "function",
      "function": {
        "name": "get_latest_statistics",
        "description": "Get latest statistics on a topic",
        "parameters": {
          "type": "object",
          "properties": {
            "topic": {"type": "string"},
            "year": {"type": "integer"}
          },
          "required": ["topic"]
        }
      }
    }
  ]
}
```

## Agentic Loop Implementation

### TypeScript Agentic Loop

```typescript
async function callLLM(messages: Message[]): Promise<ChatResponse> {
  const result = await openRouter.chat.send({
    model: '{{MODEL}}',
    tools,
    messages,
    stream: false,
  });

  messages.push(result.choices[0].message);
  return result;
}

async function getToolResponse(response: ChatResponse): Promise<Message> {
  const toolCall = response.choices[0].message.toolCalls[0];
  const toolName = toolCall.function.name;
  const toolArgs = JSON.parse(toolCall.function.arguments);

  const toolResult = await TOOL_MAPPING[toolName](toolArgs);

  return {
    role: 'tool',
    toolCallId: toolCall.id,
    content: toolResult,
  };
}

const maxIterations = 10;
let iterationCount = 0;

while (iterationCount < maxIterations) {
  iterationCount++;
  const response = await callLLM(messages);

  if (response.choices[0].message.toolCalls) {
    messages.push(await getToolResponse(response));
  } else {
    break;
  }
}

if (iterationCount >= maxIterations) {
  console.warn("Warning: Maximum iterations reached");
}

console.log(messages[messages.length - 1].content);
```

### Python Agentic Loop

```python
def call_llm(msgs):
    resp = openai_client.chat.completions.create(
        model=MODEL,
        tools=tools,
        messages=msgs
    )
    msgs.append(resp.choices[0].message.dict())
    return resp

def get_tool_response(response):
    tool_call = response.choices[0].message.tool_calls[0]
    tool_name = tool_call.function.name
    tool_args = json.loads(tool_call.function.arguments)

    tool_result = TOOL_MAPPING[tool_name](**tool_args)

    return {
        "role": "tool",
        "tool_call_id": tool_call.id,
        "content": tool_result,
    }

max_iterations = 10
iteration_count = 0

while iteration_count < max_iterations:
    iteration_count += 1
    resp = call_llm(messages)

    if resp.choices[0].message.tool_calls is not None:
        messages.append(get_tool_response(resp))
    else:
        break

if iteration_count >= max_iterations:
    print("Warning: Maximum iterations reached")

print(messages[-1]['content'])
```

## Streaming with Tool Calls

```typescript
const stream = await fetch('/api/chat/completions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    model: 'anthropic/claude-sonnet-4.5',
    messages: messages,
    tools: tools,
    stream: true
  })
});

const reader = stream.body.getReader();
let toolCalls = [];

while (true) {
  const { done, value } = await reader.read();
  if (done) {
    break;
  }

  const chunk = new TextDecoder().decode(value);
  const lines = chunk.split('\n').filter(line => line.trim());

  for (const line of lines) {
    if (line.startsWith('data: ')) {
      const data = JSON.parse(line.slice(6));

      if (data.choices[0].delta.tool_calls) {
        toolCalls.push(...data.choices[0].delta.tool_calls);
      }

      if (data.choices[0].delta.finish_reason === 'tool_calls') {
        await handleToolCalls(toolCalls);
      } else if (data.choices[0].delta.finish_reason === 'stop') {
        break;
      }
    }
  }
}
```

## JSON Request Examples

### Initial Inference Request

```json
{
  "model": "google/gemini-3-flash-preview",
  "messages": [
    {
      "role": "user",
      "content": "What are the titles of some James Joyce books?"
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "search_gutenberg_books",
        "description": "Search for books in the Project Gutenberg library",
        "parameters": {
          "type": "object",
          "properties": {
            "search_terms": {
              "type": "array",
              "items": {"type": "string"},
              "description": "List of search terms to find books"
            }
          },
          "required": ["search_terms"]
        }
      }
    }
  ]
}
```

### Request with Tool Results

```json
{
  "model": "google/gemini-3-flash-preview",
  "messages": [
    {
      "role": "user",
      "content": "What are the titles of some James Joyce books?"
    },
    {
      "role": "assistant",
      "content": null,
      "tool_calls": [
        {
          "id": "call_abc123",
          "type": "function",
          "function": {
            "name": "search_gutenberg_books",
            "arguments": "{\"search_terms\": [\"James\", \"Joyce\"]}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "tool_call_id": "call_abc123",
      "content": "[{\"id\": 4300, \"title\": \"Ulysses\", \"authors\": [{\"name\": \"Joyce, James\"}]}]"
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "search_gutenberg_books",
        "description": "Search for books in the Project Gutenberg library",
        "parameters": {
          "type": "object",
          "properties": {
            "search_terms": {
              "type": "array",
              "items": {"type": "string"},
              "description": "List of search terms to find books"
            }
          },
          "required": ["search_terms"]
        }
      }
    }
  ]
}
```

## Configuration Options

### Tool Choice

Control whether the model uses tools:

```json
{ "tool_choice": "auto" }
```

```json
{ "tool_choice": "none" }
```

Force a specific tool:

```json
{
  "tool_choice": {
    "type": "function",
    "function": {"name": "search_database"}
  }
}
```

### Parallel Tool Calls

Set to `false` to require sequential tool execution instead of simultaneous calls:

```json
{ "parallel_tool_calls": false }
```

## Multi-Tool Workflow Example

Design complementary tools that naturally chain together for intuitive multi-step workflows:

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "search_products",
        "description": "Search for products in the catalog"
      }
    },
    {
      "type": "function",
      "function": {
        "name": "get_product_details",
        "description": "Get detailed information about a specific product"
      }
    },
    {
      "type": "function",
      "function": {
        "name": "check_inventory",
        "description": "Check current inventory levels for a product"
      }
    }
  ]
}
```

## Best Practices

### Weather Tool Definition Example

Use clear, descriptive function names and comprehensive descriptions explaining when and how to use each tool. Structure parameters with explicit types, examples, and required field designations:

```json
{
  "description": "Get current weather conditions and 5-day forecast for a specific location. Supports cities, zip codes, and coordinates.",
  "parameters": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City name, zip code, or coordinates (lat,lng). Examples: 'New York', '10001', '40.7128,-74.0060'"
      },
      "units": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit preference",
        "default": "celsius"
      }
    },
    "required": ["location"]
  }
}
```
