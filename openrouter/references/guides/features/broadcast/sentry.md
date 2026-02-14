# Sentry Integration Guide

## Overview

Sentry serves as an application monitoring platform enabling developers to identify and resolve issues in real-time. The platform includes AI monitoring capabilities for tracking large language model performance and errors.

## Setup Instructions

### Step 1: Obtain Sentry Credentials

Within your Sentry project settings, retrieve two essential items:

1. Navigate to **Settings > Projects > [Your Project] > SDK Setup > Client Keys (DSN)**
2. Select the **OpenTelemetry** tab
3. Copy the **OTLP Traces Endpoint** URL (format: `.../v1/traces`)
4. Copy your **DSN** identifier

### Step 2: Activate Broadcast Feature

Navigate to [Settings > Observability](https://openrouter.ai/settings/observability) and engage the **Enable Broadcast** toggle.

### Step 3: Configure Sentry Settings

Select the edit option adjacent to **Sentry** and input:

- **OTLP Traces Endpoint**: Your Sentry OTLP URL
- **Sentry DSN**: Your project's DSN identifier

### Step 4: Validate Configuration

Execute **Test Connection** to confirm proper setup. Settings persist only upon successful testing.

### Step 5: Generate Test Trace

Submit an API request via OpenRouter and observe the resulting trace in Sentry's Performance or Traces interface.

## Custom Metadata Support

OpenTelemetry protocol enables custom metadata transmission as span attributes for filtering and analysis.

| Metadata Key | Maps To | Purpose |
|---|---|---|
| `trace_id` | Trace ID | Consolidate related requests |
| `trace_name` | Transaction Name | Label root span |
| `span_name` | Span Description | Identify intermediate spans |
| `generation_name` | Span Description | Tag LLM generation span |
| `parent_span_id` | Parent Span ID | Reference existing spans |

## Configuration Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Debug this error..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "incident_investigation_001",
    "trace_name": "Error Analysis Agent",
    "generation_name": "Analyze Stack Trace",
    "environment": "production",
    "release": "v2.1.0"
  }
}
```

## Privacy Considerations

When Privacy Mode is enabled, prompt and completion content remains excluded from trace data. Token usage, costs, timing, model details, and custom metadata transmission continues normally.
