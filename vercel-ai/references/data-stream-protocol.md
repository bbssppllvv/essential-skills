# Vercel AI SDK Data Stream Protocol

## Table of Contents

1. [Overview](#overview)
2. [SSE Format](#sse-format)
3. [Stream Part Types](#stream-part-types)
4. [Text Streaming](#text-streaming)
5. [Tool Call Streaming](#tool-call-streaming)
6. [Reasoning Streaming](#reasoning-streaming)
7. [Server Implementation](#server-implementation)
8. [Building Custom Clients (Mobile / SwiftUI)](#building-custom-clients-mobile--swiftui)
9. [Resumable Streams](#resumable-streams)

---

## Overview

The Data Stream Protocol is the wire format used by the Vercel AI SDK to stream
AI responses from server to client. It is built on top of **Server-Sent Events
(SSE)** and is the default transport for the `useChat` and `useCompletion`
React hooks.

Understanding this protocol is essential when:

- Building a **custom backend** (non-Next.js) that needs to talk to the SDK's
  frontend hooks.
- Building a **native mobile client** (SwiftUI, Kotlin, Flutter) that consumes
  the same streaming endpoint used by a web frontend.
- Implementing **resumable or fan-out streaming** in serverless environments.

### Required HTTP Headers

| Header | Value | Notes |
|--------|-------|-------|
| `Content-Type` | `text/event-stream` | Standard SSE content type |
| `x-vercel-ai-ui-message-stream` | `v1` | **Required** for custom backends so the client recognises the stream format |

Additional recommended headers for SSE:

```
Cache-Control: no-cache
Connection: keep-alive
```

---

## SSE Format

Every event in the stream is a standard SSE message. The payload is a JSON
object carried in the `data:` field:

```
data: {"type":"text-delta","id":"txt_001","delta":"Hello"}
```

Rules:

- Each logical event is a single `data:` line followed by a blank line (`\n\n`).
- The JSON object always contains a `type` field that identifies the stream
  part type.
- The stream is terminated by the sentinel value:

```
data: [DONE]
```

A minimal well-formed stream looks like this:

```
data: {"type":"message-start","id":"msg_001"}

data: {"type":"text-start","id":"txt_001"}

data: {"type":"text-delta","id":"txt_001","delta":"Hi there!"}

data: {"type":"text-end","id":"txt_001"}

data: {"type":"finish-step"}

data: {"type":"finish"}

data: [DONE]
```

---

## Stream Part Types

The complete set of part types recognised by the protocol:

| Part Type | Purpose |
|-----------|---------|
| `message-start` | Beginning of a new message. Carries message-level metadata (ID, role, etc.). |
| `text-start` | Opens a new text block. Contains a unique `id` for the block. |
| `text-delta` | An incremental chunk of text content. References the block `id`. |
| `text-end` | Closes the text block identified by `id`. |
| `reasoning-start` | Opens a reasoning / chain-of-thought block. |
| `reasoning-delta` | Incremental reasoning content. |
| `reasoning-end` | Closes the reasoning block. |
| `source-url` | An external URL reference (e.g. citation). |
| `source-document` | A document or file reference (e.g. RAG source). |
| `file` | A file attachment with media type information. |
| `data-{custom}` | Custom structured data (the suffix after `data-` is user-defined). |
| `error` | Error information. The client should surface this to the user. |
| `tool-input-start` | Opens a tool call. Contains `toolCallId` and `toolName`. |
| `tool-input-delta` | Incremental tool input (partial JSON argument string). |
| `tool-input-available` | Tool input is complete. Contains the fully parsed `args` object. |
| `tool-output-available` | Result of tool execution. Contains `output`. |
| `start-step` | Marks the beginning of a logical step (e.g. an agentic loop iteration). |
| `finish-step` | One LLM API call within the step has completed. |
| `finish` | The entire message transmission is complete. |
| `abort` | The stream was aborted (e.g. user cancellation, timeout). |

---

## Text Streaming

Text is streamed as a sequence of `text-start`, one or more `text-delta`
events, and a final `text-end`. All events in the sequence share the same `id`.

### Wire format

```
data: {"type":"text-start","id":"txt_001"}

data: {"type":"text-delta","id":"txt_001","delta":"Hello "}

data: {"type":"text-delta","id":"txt_001","delta":"world"}

data: {"type":"text-end","id":"txt_001"}
```

### Client-side accumulation

```
Buffer = ""
on text-start  -> initialise buffer for id
on text-delta  -> buffer[id] += delta
on text-end    -> finalise buffer[id], render complete text
```

A single message may contain multiple text blocks (each with a distinct `id`),
though a single block is the most common case.

---

## Tool Call Streaming

Tool calls follow a four-event lifecycle, all correlated by `toolCallId`:

### Wire format

```
data: {"type":"tool-input-start","toolCallId":"tc_001","toolName":"getWeather"}

data: {"type":"tool-input-delta","toolCallId":"tc_001","delta":"{\"city\":\"Paris\"}"}

data: {"type":"tool-input-available","toolCallId":"tc_001","args":{"city":"Paris"}}

data: {"type":"tool-output-available","toolCallId":"tc_001","output":{"temp":22}}
```

### Event lifecycle

1. **`tool-input-start`** -- Announces a new tool call. Provides the
   `toolName` so the client can begin rendering a placeholder (e.g. a
   "Searching..." indicator).
2. **`tool-input-delta`** -- Streams the raw JSON string of the tool arguments
   incrementally. Useful for showing the user what the model is "typing" into
   the tool.
3. **`tool-input-available`** -- The complete, parsed arguments are available.
   At this point the server (or client, in client-side tool execution) invokes
   the tool.
4. **`tool-output-available`** -- The tool has returned a result. The `output`
   field contains the serialised return value.

Multiple tool calls can be interleaved in a single message; always match events
by `toolCallId`.

---

## Reasoning Streaming

When a model exposes chain-of-thought or reasoning tokens (e.g. extended
thinking), they are streamed with `reasoning-*` events:

```
data: {"type":"reasoning-start","id":"reason_001"}

data: {"type":"reasoning-delta","id":"reason_001","delta":"Let me think about this..."}

data: {"type":"reasoning-delta","id":"reason_001","delta":" The user wants weather data."}

data: {"type":"reasoning-end","id":"reason_001"}
```

Reasoning blocks typically appear before the corresponding text blocks within
the same step. Clients can choose to display them in a collapsible "thinking"
UI or hide them entirely.

---

## Server Implementation

### Method 1: Using `streamText` (recommended)

The simplest approach -- call `streamText` and convert the result directly:

```typescript
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages,
  });

  // Converts the stream into a proper SSE response with all
  // required headers, including x-vercel-ai-ui-message-stream.
  return result.toUIMessageStreamResponse();
}
```

### Method 2: Custom stream with manual control

For advanced scenarios where you need to inject custom events, merge multiple
streams, or add middleware:

```typescript
import {
  createUIMessageStream,
  createUIMessageStreamResponse,
} from 'ai';

export async function POST(req: Request) {
  const stream = createUIMessageStream({
    execute: async ({ writer }) => {
      // Write events manually
      writer.write({ type: 'text-start', id: 'txt_001' });
      writer.write({ type: 'text-delta', id: 'txt_001', delta: 'Hello' });
      writer.write({ type: 'text-end', id: 'txt_001' });
      writer.write({ type: 'finish' });
    },
  });

  return createUIMessageStreamResponse({ stream });
}
```

### Method 3: Merging with an upstream AI SDK stream

```typescript
import {
  createUIMessageStream,
  createUIMessageStreamResponse,
  streamText,
} from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const stream = createUIMessageStream({
    execute: async ({ writer, merge }) => {
      const result = streamText({
        model: openai('gpt-4o'),
        messages,
      });

      // Merge the AI stream into the custom stream
      merge(result.toUIMessageStream());

      // You can still write additional custom events
      writer.write({
        type: 'data-custom-metadata',
        data: { requestId: crypto.randomUUID() },
      });
    },
  });

  return createUIMessageStreamResponse({ stream });
}
```

---

## Building Custom Clients (Mobile / SwiftUI)

Native mobile clients cannot use the React hooks (`useChat`, `useCompletion`).
Instead, they must parse the SSE stream directly. The following steps describe
the general approach, with a SwiftUI pseudocode example.

### General approach

1. **Open an HTTP connection** with the required headers
   (`Content-Type: text/event-stream`, `x-vercel-ai-ui-message-stream: v1`).
2. **Read the response line by line.** Each SSE event starts with `data: `.
3. **Strip the `data: ` prefix** and check for the `[DONE]` sentinel.
4. **Parse the JSON** payload from each data line.
5. **Switch on the `type` field** to dispatch to the appropriate handler.
6. **Accumulate text deltas** into a running text buffer, keyed by `id`.
7. **Handle tool calls** by matching `toolCallId` across the four tool events.
8. **Watch for `finish`** to know the message is complete.
9. **Watch for `abort`** to handle cancellation gracefully.

### SwiftUI pseudocode

```swift
import Foundation

// MARK: - Stream Part Model

enum StreamPartType: String, Codable {
    case messageStart    = "message-start"
    case textStart       = "text-start"
    case textDelta       = "text-delta"
    case textEnd         = "text-end"
    case reasoningStart  = "reasoning-start"
    case reasoningDelta  = "reasoning-delta"
    case reasoningEnd    = "reasoning-end"
    case toolInputStart  = "tool-input-start"
    case toolInputDelta  = "tool-input-delta"
    case toolInputAvailable = "tool-input-available"
    case toolOutputAvailable = "tool-output-available"
    case startStep       = "start-step"
    case finishStep      = "finish-step"
    case finish          = "finish"
    case abort           = "abort"
    case error           = "error"
}

struct StreamPart: Codable {
    let type: StreamPartType
    let id: String?
    let delta: String?
    let toolCallId: String?
    let toolName: String?
    // Additional fields omitted for brevity
}

// MARK: - SSE Parser

class AIStreamParser: NSObject, URLSessionDataDelegate {
    private var buffer = ""
    private var textBuffers: [String: String] = [:]
    var onTextUpdate: ((String) -> Void)?
    var onComplete: (() -> Void)?

    func startStreaming(url: URL, messages: [[String: Any]]) {
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("text/event-stream", forHTTPHeaderField: "Accept")
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try? JSONSerialization.data(
            withJSONObject: ["messages": messages]
        )

        let session = URLSession(
            configuration: .default,
            delegate: self,
            delegateQueue: .main
        )
        session.dataTask(with: request).resume()
    }

    func urlSession(
        _ session: URLSession,
        dataTask: URLSessionDataTask,
        didReceive data: Data
    ) {
        guard let text = String(data: data, encoding: .utf8) else { return }
        buffer += text

        // Process complete lines
        while let newlineRange = buffer.range(of: "\n") {
            let line = String(buffer[buffer.startIndex..<newlineRange.lowerBound])
            buffer = String(buffer[newlineRange.upperBound...])
            processLine(line)
        }
    }

    private func processLine(_ line: String) {
        // Skip empty lines and comments
        guard line.hasPrefix("data: ") else { return }
        let payload = String(line.dropFirst(6))

        // Check for stream termination
        if payload == "[DONE]" {
            onComplete?()
            return
        }

        // Parse JSON
        guard let data = payload.data(using: .utf8),
              let part = try? JSONDecoder().decode(StreamPart.self, from: data)
        else { return }

        // Dispatch on type
        switch part.type {
        case .textStart:
            if let id = part.id {
                textBuffers[id] = ""
            }

        case .textDelta:
            if let id = part.id, let delta = part.delta {
                textBuffers[id, default: ""] += delta
                let fullText = textBuffers.values.joined()
                onTextUpdate?(fullText)
            }

        case .textEnd:
            break // Text block finalised

        case .toolInputStart:
            // Show loading indicator for tool
            break

        case .toolOutputAvailable:
            // Display tool result
            break

        case .finish:
            onComplete?()

        case .abort:
            // Handle cancellation
            break

        case .error:
            // Surface error to user
            break

        default:
            break
        }
    }
}

// MARK: - Usage in a SwiftUI ViewModel

@MainActor
class ChatViewModel: ObservableObject {
    @Published var responseText = ""
    @Published var isStreaming = false
    private let parser = AIStreamParser()

    func sendMessage(_ content: String) {
        isStreaming = true
        responseText = ""

        parser.onTextUpdate = { [weak self] text in
            self?.responseText = text
        }
        parser.onComplete = { [weak self] in
            self?.isStreaming = false
        }

        let messages = [["role": "user", "content": content]]
        parser.startStreaming(
            url: URL(string: "https://your-api.com/api/chat")!,
            messages: messages
        )
    }
}
```

### Key considerations for mobile clients

- **Backpressure**: Mobile networks can be slow. Buffer incoming data and
  process it in batches if UI updates become a bottleneck.
- **Reconnection**: SSE does not automatically reconnect on mobile. Implement
  your own retry logic with exponential backoff.
- **Background handling**: On iOS, long-running streams may be suspended when
  the app moves to background. Use `beginBackgroundTask` for short grace
  periods, or accept that the stream will be interrupted.
- **Partial JSON**: `tool-input-delta` events contain partial JSON strings.
  Do not attempt to parse them as JSON until `tool-input-available` arrives.

---

## Resumable Streams

The `resumable-stream` package (524+ GitHub stars) solves a critical problem in
serverless environments: if a client disconnects mid-stream (e.g. a Vercel
function cold start, network blip, or tab switch), the server-side stream
producer keeps running but the data is lost.

### How it works

1. The **producer** writes every stream chunk to a **Redis pub/sub channel**
   and simultaneously appends it to a **Redis list** (the buffer).
2. The **first consumer** subscribes to the pub/sub channel and receives
   chunks in real time.
3. If the consumer disconnects and **reconnects**, a new consumer is created
   that:
   - Reads all buffered chunks from the Redis list (replay).
   - Subscribes to the pub/sub channel for any new chunks.
   - Deduplicates so no chunk is delivered twice.
4. The producer **always runs to completion**, regardless of consumer state.

### Supported Redis implementations

- `redis` (Node Redis)
- `ioredis`
- Upstash Redis (HTTP-based, ideal for edge/serverless)
- Valkey

### Usage pattern

```typescript
import { createResumableStream } from 'resumable-stream';

// On the server, wrap your AI stream:
const resumable = createResumableStream({
  redis,
  streamId: `chat:${chatId}`,
});

// Pipe the AI SDK stream into the resumable producer
result.toUIMessageStream().pipeTo(resumable.writable);

// Return the resumable readable to the client
return new Response(resumable.readable, {
  headers: {
    'Content-Type': 'text/event-stream',
    'x-vercel-ai-ui-message-stream': 'v1',
  },
});
```

### When to use resumable streams

- **Serverless deployments** where functions may be recycled mid-stream.
- **Long-running generations** (e.g. agentic loops with multiple tool calls)
  where the risk of disconnection is high.
- **Fan-out** scenarios where multiple clients need to consume the same stream.

The Vercel AI Chatbot template (`vercel/ai-chatbot`) uses `resumable-stream`
by default.
