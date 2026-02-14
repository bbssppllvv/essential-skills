---
title: Broadcast
description: Send traces to external observability and analytics platforms
---

# Broadcast

Broadcast allows you to automatically send traces from your OpenRouter requests to external observability and analytics platforms without requiring additional application code instrumentation.

## Getting Started

Enable the feature by navigating to **Settings > Observability** in your OpenRouter dashboard and toggling the broadcast switch.

> **Note:** Organization admins must authorize this feature for team accounts.

## Supported Destinations

| Destination | Status |
|---|---|
| Arize AI | Active |
| Braintrust | Active |
| ClickHouse | Active |
| Comet Opik | Active |
| Datadog | Active |
| Grafana Cloud | Active |
| Langfuse | Active |
| LangSmith | Active |
| New Relic | Active |
| OpenTelemetry Collector | Active |
| PostHog | Active |
| S3 / S3-Compatible | Active |
| Sentry | Active |
| Snowflake | Active |
| W&B Weave | Active |
| Webhook | Active |

### Coming Soon

- AWS Firehose
- Dynatrace
- Evidently
- Fiddler
- Galileo
- Helicone
- HoneyHive
- Keywords AI
- Middleware
- Mona
- OpenInference
- Phoenix
- Portkey
- Supabase
- WhyLabs

## Trace Data

Each trace captures:

- Request input and model output
- Token counts (prompt, completion, total)
- Cost information
- Timing metrics and latency
- Model and provider details
- Tool usage information

## Custom Fields

You can enrich traces using optional parameters in your API requests.

### User

Associate traces with specific end-users (max 128 characters):

```json
{
  "model": "openai/gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": "Hello, world!"
    }
  ],
  "user": "user_12345"
}
```

### Session ID

Group related requests together (max 128 characters). Also accepts the `x-session-id` HTTP header:

```json
{
  "model": "openai/gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": "Hello, world!"
    }
  ],
  "session_id": "session_abc123"
}
```

### Trace Metadata

The `trace` field accepts arbitrary JSON metadata for organizing and filtering traces:

```json
{
  "model": "openai/gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": "Summarize this document..."
    }
  ],
  "trace": {
    "trace_id": "workflow_12345",
    "trace_name": "Document Processing",
    "span_name": "Summarization Step",
    "generation_name": "Generate Summary",
    "environment": "production",
    "feature": "customer-support",
    "version": "1.2.3"
  }
}
```

### Supported Metadata Keys

| Key | Description |
|---|---|
| `trace_id` | Group multiple API requests into a single trace. Use the same ID across requests to track multi-step workflows. |
| `trace_name` | Custom name for the root trace in your observability platform. Defaults to the model name if not set. |
| `span_name` | Create a parent span that groups LLM operations. Creates hierarchical structure where the span contains the generation. |
| `generation_name` | Custom name for the specific LLM generation/call. Defaults to the model name if not set. |
| `parent_span_id` | Link your OpenRouter trace to an existing span from your own tracing system (e.g., OpenTelemetry). |

Any additional keys in the `trace` object (like `environment`, `feature`, `version` in the example above) are passed as custom metadata to your observability platform.

### Linking to Existing Traces

You can link OpenRouter traces to your existing tracing system:

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Hello!" }],
  "trace": {
    "trace_id": "your-existing-trace-id",
    "parent_span_id": "your-existing-span-id"
  }
}
```

## Advanced Features

### Sampling Rate

Each destination supports configurable sampling to manage data volume and costs. Sampling is deterministic -- when a `session_id` is provided, all traces within that session are consistently included or excluded together, ensuring complete session visibility rather than fragmented data across observability platforms.

### Privacy Mode

When Privacy Mode is enabled, the following data is stripped before sending traces:

- Input messages
- Output choices

Token counts, costs, timing, model information, and custom metadata remain intact. This enables monitoring usage metrics while protecting conversation content for compliance purposes.

### API Key Filtering

Each destination can be restricted to specific API keys, allowing teams to route traces selectively across platforms, isolate particular use cases, or apply different sampling strategies to production versus development keys. When no API keys are selected, the destination receives all traces.

### Organization Support

Broadcast functions at both individual and organizational levels. Organization administrators can establish shared destinations applied across all organizational API keys, promoting consistent observability practices throughout teams.

### Security

Credentials are encrypted and stored securely. Trace transmission occurs asynchronously without adding latency to your API requests.
