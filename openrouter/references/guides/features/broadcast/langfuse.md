---
title: Broadcast to Langfuse
description: Send OpenRouter traces to Langfuse for LLM observability
---

# Broadcast to Langfuse

Langfuse is an open-source LLM engineering platform for tracing, evaluating, and debugging LLM applications. With OpenRouter's Broadcast feature, traces are automatically sent to Langfuse without additional instrumentation.

## Setup

### Step 1: Get Your Langfuse API Keys

Generate a **Secret Key** and **Public Key** pair in Langfuse's **Settings > API Keys** section.

### Step 2: Enable Broadcast

Navigate to **Settings > Observability** in OpenRouter and activate the Broadcast feature.

### Step 3: Configure Langfuse Destination

Add Langfuse as a destination and provide:

- **Secret Key**: Your Langfuse secret key
- **Public Key**: Your Langfuse public key
- **Base URL** (optional): Defaults to `https://us.cloud.langfuse.com`. Update for regional or self-hosted deployments.

### Step 4: Test Connection

Click **Test Connection** to verify the setup. Configuration saves only upon successful validation.

### Step 5: Verify Traces

Send an API request through OpenRouter and confirm the trace appears in Langfuse.

## Custom Metadata

Enrich your traces with custom metadata by including the `trace` field in your API requests:

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Summarize this document..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "workflow_12345",
    "trace_name": "Document Processing Pipeline",
    "span_name": "Summarization Step",
    "generation_name": "Generate Summary",
    "environment": "production",
    "pipeline_version": "2.1.0"
  }
}
```

### Trace Hierarchy

The metadata creates a hierarchical structure in Langfuse:

```
Document Processing Pipeline (trace)
└── Summarization Step (span)
    └── Generate Summary (generation)
```

### Metadata Mapping

| Key | Langfuse Mapping | Description |
|---|---|---|
| `trace_id` | Trace ID | Group multiple requests into a single trace |
| `trace_name` | Trace Name | Custom name displayed in the Langfuse trace list |
| `span_name` | Span Name | Name for intermediate spans in the hierarchy |
| `generation_name` | Generation Name | Name for the LLM generation observation |
| `parent_span_id` | Parent Observation ID | Link to an existing span in your trace hierarchy |

### Additional Context Fields

- The `user` field maps to Langfuse **User ID** for analytics
- The `session_id` field maps to Langfuse **Session ID** for conversation grouping
- Extra metadata keys within `trace` are available for filtering in Langfuse

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. Token usage, costs, timing, model information, and custom metadata remain transmitted.
