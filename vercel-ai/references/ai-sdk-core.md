# AI SDK Core Reference

## Table of Contents
1. [generateText](#generatetext)
2. [streamText](#streamtext)
3. [Output Helpers](#output-helpers)
4. [Provider System](#provider-system)
5. [Embeddings](#embeddings)
6. [Image, Speech & Transcription](#image-speech--transcription)
7. [Error Handling](#error-handling)
8. [New v6 APIs](#new-v6-apis)
9. [Provider-Specific Features](#provider-specific-features)
10. [Legacy: generateObject & streamObject](#legacy-generateobject--streamobject)

---

## generateText

Non-streaming text generation for background tasks, scripts, and non-interactive use cases.

```typescript
import { generateText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const { text, usage, finishReason, steps } = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  prompt: 'Explain quantum computing in one paragraph.',
  // OR use messages array:
  // messages: [{ role: 'user', content: 'Hello' }],
  maxTokens: 1000,
  temperature: 0.7,
  maxRetries: 2, // default
});
```

### Key parameters
- `model` — Provider model instance (required)
- `prompt` or `messages` — Input (required, one of them)
- `system` — System prompt
- `maxTokens` — Max output tokens
- `temperature` — 0-1 randomness
- `topP`, `topK` — Sampling parameters
- `maxRetries` — Retry count (default: 2)
- `abortSignal` — Cancellation signal
- `tools` — Tool definitions (enables tool calling)
- `stopWhen` — Stop condition, e.g. `stepCountIs(5)` (replaces removed `maxSteps`)
- `output` — Output helper for structured output
- `activeTools` — Subset of tools available per step
- `toolChoice` — `'auto' | 'required' | 'none' | { type: 'tool', toolName: string }`

### Return value
- `text` — Generated text
- `usage` — `{ promptTokens, completionTokens, totalTokens }`
- `finishReason` — `'stop' | 'length' | 'content-filter' | 'tool-calls' | 'error'`
- `steps` — Array of step details (for multi-step tool use)
- `toolCalls` — Tool calls made
- `toolResults` — Results from tool execution
- `response` — Raw response metadata

---

## streamText

Streaming text generation for real-time interactive responses.

```typescript
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

const result = streamText({
  model: openai('gpt-4o'),
  messages,
  onChunk: ({ chunk }) => { /* process each chunk */ },
  onFinish: ({ text, usage }) => { /* finalization logic */ },
});

// For Next.js API routes:
return result.toDataStreamResponse();

// For useChat integration:
return result.toUIMessageStreamResponse();

// For plain text streaming:
return result.toTextStreamResponse();
```

### Consuming the stream
```typescript
// Async iteration
for await (const textPart of result.textStream) {
  process.stdout.write(textPart);
}

// Full text (awaitable)
const fullText = await result.text;

// Usage (awaitable)
const usage = await result.usage;
```

### Callbacks
- `onChunk({ chunk })` — Called for each streaming chunk
- `onFinish({ text, usage, finishReason, steps })` — Called when streaming completes
- `onStepFinish({ text, toolCalls, toolResults })` — Called after each step in multi-step

---

## Output Helpers

AI SDK v6 unifies structured output with `generateText`/`streamText` via Output helpers. This allows combining tool calling with structured output in a single request.

```typescript
import { streamText, Output, stepCountIs } from 'ai';
import { z } from 'zod';

// Object output
const result = streamText({
  model: anthropic('claude-sonnet-4-5'),
  messages,
  tools: { /* ... */ },
  output: Output.object({
    schema: z.object({
      summary: z.string(),
      confidence: z.number(),
    }),
  }),
  stopWhen: stepCountIs(5),
});

// Array output with element streaming
const result = streamText({
  model: openai('gpt-4o'),
  prompt: 'List 5 books',
  output: Output.array({
    element: z.object({ title: z.string(), author: z.string() }),
  }),
});
for await (const element of result.elementStream) {
  console.log(element); // Each complete element as it's generated
}

// Choice output (classification)
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  prompt: 'Is this positive or negative: "I love this!"',
  output: Output.choice({ options: ['positive', 'negative', 'neutral'] }),
});

// Unstructured JSON (no schema validation)
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  prompt: 'Return some JSON about this topic',
  output: Output.json(),
});

// Plain text
const result = streamText({
  output: Output.text(),
  // ...
});
```

---

## Provider System

All providers follow a unified interface. Install provider packages separately.

### Available providers

| Provider | Package | Model example |
|----------|---------|--------------|
| Anthropic | `@ai-sdk/anthropic` | `anthropic('claude-sonnet-4-5')` |
| OpenAI | `@ai-sdk/openai` | `openai('gpt-4o')` |
| Google | `@ai-sdk/google` | `google('gemini-2.0-flash')` |
| Mistral | `@ai-sdk/mistral` | `mistral('mistral-large-latest')` |
| Cohere | `@ai-sdk/cohere` | `cohere('command-r-plus')` |
| Amazon Bedrock | `@ai-sdk/amazon-bedrock` | `bedrock('anthropic.claude-3-sonnet')` |
| Azure OpenAI | `@ai-sdk/azure` | `azure('gpt-4o')` |
| Fireworks | `@ai-sdk/fireworks` | `fireworks('accounts/fireworks/models/llama-v3p1-70b')` |
| Groq | `@ai-sdk/groq` | `groq('llama-3.1-70b-versatile')` |
| AI Gateway | `@ai-sdk/gateway` | `gateway('anthropic/claude-sonnet-4.5')` |
| xAI | `@ai-sdk/xai` | `xai('grok-2')` |
| DeepSeek | `@ai-sdk/deepseek` | `deepseek('deepseek-chat')` |
| Together.ai | `@ai-sdk/togetherai` | `togetherai('meta-llama/...')` |
| Perplexity | `@ai-sdk/perplexity` | `perplexity('llama-3.1-sonar-large')` |
| ElevenLabs | `@ai-sdk/elevenlabs` | Speech/voice provider |
| Cerebras | `@ai-sdk/cerebras` | `cerebras('llama3.1-70b')` |

35+ providers total in the AI SDK monorepo including Alibaba, ByteDance, Replicate, HuggingFace, and many more community providers.

### Provider registry

```typescript
import { createProviderRegistry } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';
import { openai } from '@ai-sdk/openai';

const registry = createProviderRegistry({
  anthropic,
  openai,
});

// Use with string model IDs
const model = registry.languageModel('anthropic:claude-sonnet-4-5');
```

### Custom provider (defaults/aliases)

```typescript
import { customProvider } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const myProvider = customProvider({
  languageModels: {
    fast: anthropic('claude-haiku-4-5'),
    smart: anthropic('claude-sonnet-4-5'),
  },
});

const result = await generateText({
  model: myProvider('smart'),
  prompt: 'Hello',
});
```

---

## Embeddings

```typescript
import { embed, embedMany, cosineSimilarity } from 'ai';
import { openai } from '@ai-sdk/openai';

// Single embedding
const { embedding } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'Search query text',
});

// Batch embeddings (for RAG ingestion)
const { embeddings } = await embedMany({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  values: ['doc chunk 1', 'doc chunk 2', 'doc chunk 3'],
  maxParallelCalls: 5, // Parallel processing
});

// Similarity comparison
const similarity = cosineSimilarity(embedding1, embedding2);
// Returns -1 to 1 (higher = more similar)
```

### RAG pattern

```typescript
// 1. Embed and store documents
const { embeddings } = await embedMany({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  values: documentChunks,
});
// Store embeddings in vector DB (Pinecone, Upstash, etc.)

// 2. Query: embed the question, find similar chunks
const { embedding } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: userQuestion,
});
const relevantChunks = await vectorDB.query(embedding, { topK: 5 });

// 3. Generate answer with context
const { text } = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  system: `Answer based on this context:\n${relevantChunks.join('\n')}`,
  prompt: userQuestion,
});
```

---

## Image, Speech & Transcription

### generateImage

```typescript
import { generateImage } from 'ai';
import { openai } from '@ai-sdk/openai';

const { image } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'A futuristic city skyline at sunset',
  size: '1024x1024',
});
// image.base64 — base64-encoded image data
```

### generateSpeech

```typescript
import { generateSpeech } from 'ai';
import { openai } from '@ai-sdk/openai';

const { audio } = await generateSpeech({
  model: openai.speech('tts-1'),
  text: 'Hello, welcome to our application!',
  voice: 'alloy',
});
// audio — audio data (Uint8Array)
```

### transcribe

```typescript
import { transcribe } from 'ai';
import { openai } from '@ai-sdk/openai';

const { text } = await transcribe({
  model: openai.transcription('whisper-1'),
  audio: audioFile, // File, Blob, or URL
});
```

### smoothStream

Utility for smoothing streamed text output to reduce visual chunkiness:

```typescript
import { streamText, smoothStream } from 'ai';

const result = streamText({
  model: anthropic('claude-sonnet-4-5'),
  prompt: 'Write a story',
  experimental_transform: smoothStream(),
});
```

---

## Error Handling

### Error types

```typescript
import { AI_APICallError, AI_InvalidPromptError } from 'ai';

try {
  const result = await generateText({ /* ... */ });
} catch (error) {
  if (AI_APICallError.isInstance(error)) {
    console.log(error.statusCode);    // HTTP status
    console.log(error.message);       // Error message
    console.log(error.isRetryable);   // Can retry?
    console.log(error.data);          // Additional data
  }
  if (AI_InvalidPromptError.isInstance(error)) {
    console.log(error.message);       // What's wrong with the prompt
  }
}
```

### Retry configuration

```typescript
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  maxRetries: 3, // Default: 2. Uses exponential backoff.
  prompt: 'Hello',
});
```

### Key error types
- `AI_APICallError` — API call failed (network, auth, rate limit)
- `AI_RetryError` — All retries exhausted
- `AI_InvalidPromptError` — Malformed prompt
- `AI_NoSuchModelError` — Model not found
- `AI_LoadAPIKeyError` — Missing API key

---

## New v6 APIs

### rerank

Document reranking for improved RAG relevance:

```typescript
import { rerank } from 'ai';
import { cohere } from '@ai-sdk/cohere';

const { results } = await rerank({
  model: cohere.reranker('rerank-v3.5'),
  query: 'What is machine learning?',
  documents: ['ML is a subset of AI...', 'Cooking recipes...', 'Neural networks...'],
  topK: 3,
});
// results: [{ document, score, index }] sorted by relevance
```

Supported providers: Cohere, Amazon Bedrock, Together.ai.

### generateVideo

```typescript
import { generateVideo } from 'ai';

const { video } = await generateVideo({
  model: provider.videoModel('model-id'),
  prompt: 'A sunset over mountains',
});
```

### dynamicTool

Runtime-defined tools with dynamic schemas:

```typescript
import { dynamicTool } from 'ai';

const tool = dynamicTool({
  description: 'Dynamic tool',
  inputSchema: dynamicallyGeneratedSchema,
  execute: async (args) => { /* ... */ },
});
```

### pruneMessages

Utility for managing message history length:

```typescript
import { pruneMessages } from 'ai';

const trimmed = pruneMessages(messages, { maxTokens: 4000 });
```

### devToolsMiddleware

```typescript
import { wrapLanguageModel, devToolsMiddleware } from 'ai';

const model = wrapLanguageModel({
  model: anthropic('claude-sonnet-4-5'),
  middleware: [devToolsMiddleware()],
});
// Then run: npx @ai-sdk/devtools to inspect calls
```

---

## Provider-Specific Features

### Anthropic (`@ai-sdk/anthropic`)

**Prompt Caching**:
```typescript
const result = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  messages: [{
    role: 'user',
    content: 'Long text...',
    providerOptions: {
      anthropic: { cacheControl: { type: 'ephemeral' } }
      // Extended TTL: { type: 'ephemeral', ttl: '1h' }
    }
  }],
});
// Cache stats: result.providerMetadata.anthropic.cacheCreationInputTokens
```
Minimum cacheable: 4096 tokens (Opus), 2048 tokens (Haiku).

**Extended Thinking**:
```typescript
const result = await generateText({
  model: anthropic('claude-sonnet-4-5', {
    thinking: { type: 'enabled', budgetTokens: 12000 }
  }),
  prompt: 'Complex problem...',
});
// result.reasoningText, result.reasoning
```

**Effort & Speed**: `effort: 'high'|'medium'|'low'` (Opus 4.5+), `speed: 'fast'|'standard'` (Opus 4.6)

**Context Management**: `compact_20260112` for automatic context summarization in long-running agents.

**Built-in Tools**: bash, memory, textEditor, computer, webSearch, webFetch, codeExecution, toolSearchBm25

**structuredOutputMode**: `'outputFormat'` | `'jsonTool'` | `'auto'` (Sonnet 4.5+)

**Important**: UIMessage does not support providerOptions — use `convertToModelMessages()` first.

### OpenAI (`@ai-sdk/openai`)

**Dual API**: Responses API (default) and Chat Completions (`openai.chat()`).

**Strict Mode**: `strictJsonSchema` defaults to `true` in v6. Schema restriction: no `.optional()`, use `.nullable()`.

**Prompt Caching**: Auto-enabled for 1024+ tokens. Extended retention: `promptCacheRetention: '24h'` (GPT-5.1).

**Reasoning Models** (o1, o3, o4): `reasoningEffort`, `reasoningSummary` options.

**Built-in Tools**: webSearch, fileSearch, imageGeneration, codeInterpreter, mcp, shell, applyPatch

**WebSocket Transport**: `ai-sdk-openai-websocket-fetch` for reduced TTFB in agentic workflows.

**Image Generation**: DALL-E 2/3, gpt-image-1 (inpainting, background removal, multi-image).

### Google (`@ai-sdk/google` / `@ai-sdk/google-vertex`)

**Built-in Tools**: Google Maps grounding, Vertex RAG Store, File Search.

**v6 change**: Vertex metadata key changed from `google` to `vertex`.

---

## Legacy: generateObject & streamObject

> **Deprecated in v6.** Use `generateText`/`streamText` with the `output` parameter instead (see Output Helpers above). These functions still work but will be removed in a future version.

```typescript
import { generateObject } from 'ai';
import { z } from 'zod';

// Deprecated — use generateText + Output.object() instead
const { object } = await generateObject({
  model: anthropic('claude-sonnet-4-5'),
  schema: z.object({
    recipe: z.object({
      name: z.string(),
      ingredients: z.array(z.string()),
    }),
  }),
  prompt: 'Generate a pasta recipe.',
});
```
