# New Relic Broadcast Integration

## Overview

"New Relic is a full-stack observability platform for monitoring applications, infrastructure, and digital experiences." This integration enables automatic trace delivery to New Relic from OpenRouter requests.

## Setup Instructions

### Step 1: Obtain License Key
Access your New Relic account settings and navigate to API Keys. Create a new Ingest - License key or retrieve an existing one for authentication.

### Step 2: Enable Broadcast Feature
Navigate to Settings > Observability in OpenRouter and activate the Broadcast toggle.

### Step 3: Configure New Relic Integration
Select the New Relic edit option and provide:
- Your New Relic ingest license key
- Your region preference (US or EU)

### Step 4: Verify Configuration
Use the Test Connection feature to validate setup. Configuration saves only upon successful testing.

### Step 5: Transmit Test Trace
Execute an API request via OpenRouter and confirm trace visibility in New Relic's distributed tracing interface.

## Custom Metadata Support

Traces reach New Relic through the OTLP protocol. Custom metadata from the `trace` field appears as span attributes.

| Metadata Key | New Relic Field | Purpose |
|---|---|---|
| `trace_id` | Trace ID | Consolidate multiple requests |
| `trace_name` | Span Name | Root span identifier |
| `span_name` | Span Name | Intermediate span identifier |
| `generation_name` | Span Name | LLM generation span identifier |
| `parent_span_id` | Parent Span ID | Reference existing spans |

## Privacy Mode

When Privacy Mode is enabled, "prompt and completion content is excluded from traces." Token usage, costs, timing, model information, and custom metadata remain transmitted normally.
