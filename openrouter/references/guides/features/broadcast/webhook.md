# Webhook Documentation

## Overview
OpenRouter's webhook feature enables sending traces to any HTTP endpoint accepting JSON payloads. This integration supports custom observability systems, internal monitoring, and OTLP-compatible platforms.

## Setup Process

### Endpoint Requirements
Your HTTP endpoint must:
- Accept `application/json` content
- Return 2xx status codes (400 also acceptable for empty payloads)
- Be publicly internet-accessible
- Process POST or PUT requests

### Configuration Steps
1. Navigate to Settings > Observability and enable Broadcast
2. Click the webhook edit icon and provide:
   - **URL**: Your endpoint address
   - **Method**: POST (default) or PUT
   - **Headers**: Optional JSON object for auth/security

### Authentication Example
```json
{
  "Authorization": "Bearer your-token",
  "X-Webhook-Signature": "your-webhook-secret"
}
```

### Testing
Select "Test Connection" to verify setup. OpenRouter sends an empty OTLP payload with an `X-Test-Connection: true` header.

## Data Format

Traces arrive in OpenTelemetry Protocol (OTLP) JSON format containing:
- Trace and span identifiers
- Timing data and duration
- Model and provider details
- Token consumption and costs
- Request/response content (multimodal content excluded)

### Sample Payload Structure
```json
{
  "resourceSpans": [{
    "resource": {
      "attributes": [
        { "key": "service.name", "value": { "stringValue": "openrouter" } }
      ]
    },
    "scopeSpans": [{
      "spans": [{
        "traceId": "abc123...",
        "spanId": "def456...",
        "name": "chat",
        "startTimeUnixNano": "1705312800000000000",
        "endTimeUnixNano": "1705312801000000000",
        "attributes": [
          { "key": "gen_ai.request.model", "value": { "stringValue": "openai/gpt-4" } },
          { "key": "gen_ai.usage.prompt_tokens", "value": { "intValue": "100" } }
        ]
      }]
    }]
  }]
}
```

## Custom Metadata Support

The `trace` field supports structured metadata mapping to OTLP span attributes:

| Metadata Key | OTLP Field | Purpose |
|---|---|---|
| `trace_id` | traceId | Group related requests |
| `trace_name` | Span name | Root span identifier |
| `generation_name` | Span name | LLM generation label |
| `parent_span_id` | parentSpanId | Establish span relationships |

Custom metadata appears as `trace.metadata.*` attributes in the payload.

## Use Cases
- Custom analytics and data warehouse integration
- Proprietary observability platform connection
- Event-driven workflow triggers
- Compliance and regulatory logging
- Development/testing via services like webhook.site

## Privacy Considerations
When Privacy Mode is enabled, prompt and completion text is excluded from traces. Token usage, costs, timing, model information, and metadata remain included.
