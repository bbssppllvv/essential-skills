---
title: Broadcast to Datadog
description: Send OpenRouter traces to Datadog LLM Observability
---

# Broadcast to Datadog

With Datadog LLM Observability, you can investigate the root cause of issues, monitor operational performance, and evaluate the quality, privacy, and safety of your LLM applications. With OpenRouter's Broadcast feature, traces are automatically sent to Datadog without additional instrumentation.

## Setup

### Step 1: Get Your Datadog API Key

Go to **Organization Settings > API Keys** in your Datadog dashboard and create a new key.

### Step 2: Enable Broadcast

Navigate to **Settings > Observability** in OpenRouter and activate the Broadcast feature.

### Step 3: Configure Datadog Destination

Add Datadog as a destination and provide:

- **Api Key**: Your Datadog API key
- **Ml App**: Application identifier (e.g., `production-app`)
- **Url** (optional): Datadog endpoint URL. Defaults to `https://api.us5.datadoghq.com`. Update for your region.

### Step 4: Test Connection

Click **Test Connection** to verify the setup. Configuration saves only upon successful validation.

### Step 5: Verify Traces

Send an API request through OpenRouter and confirm the trace appears in Datadog LLM Observability.

## Custom Metadata

Enrich your traces with custom metadata by including the `trace` field in your API requests:

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Hello!" }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_name": "Customer Support Bot",
    "environment": "production",
    "team": "support",
    "ticket_id": "TICKET-1234"
  }
}
```

### Metadata Mapping

| Key | Datadog Mapping | Description |
|---|---|---|
| `trace_id` | Trace ID | Group multiple requests into a single trace |
| `trace_name` | Span Name | Custom name for the root span |
| `span_name` | Span Name | Name for intermediate workflow spans |
| `generation_name` | Span Name | Name for the LLM span |

### Auto-Generated Tags

The following tags are automatically added to traces:

- `service:{ml_app}` -- derived from the ML App Name configured in the destination
- `user_id:{user}` -- derived from the `user` field in the request

Additional metadata keys in the `trace` object appear as searchable metadata within Datadog's span details.

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. Token usage, timing, costs, and custom metadata remain visible in Datadog.
