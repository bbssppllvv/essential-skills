# OpenTelemetry Collector Broadcast Documentation

## Overview

"OpenTelemetry is an open-source observability framework for collecting, processing, and exporting telemetry data." OpenRouter integrates with any OTLP-compatible backend like Axiom, Jaeger, Grafana Tempo, and self-hosted collectors.

## Setup Steps

### Step 1: Obtain OTLP Credentials

Gather your backend's OTLP endpoint URL and authentication details.

**For Axiom:**
- Create account and dataset
- Generate API token from Settings > API Tokens
- Endpoint: `https://api.axiom.co/v1/traces`
- Required headers: Authorization bearer token and dataset name

**For Self-Hosted Collectors:**
- Deploy OpenTelemetry Collector with OTLP receiver
- Configure publicly accessible endpoint
- Note the endpoint URL (typically `/v1/traces`)

### Step 2: Enable Broadcast

Navigate to Settings > Observability and activate the broadcast feature.

### Step 3: Configure the Destination

Edit the OpenTelemetry Collector settings with:
- **Endpoint URL** (e.g., `https://api.axiom.co/v1/traces`)
- **Headers** as JSON object for authentication

Example header format:
```json
{
  "Authorization": "Bearer token-value",
  "X-Custom-Header": "value"
}
```

### Step 4: Validate Configuration

"Click Test Connection to verify the setup. The configuration only saves if the test passes."

### Step 5: Generate Test Traces

Make an API request through OpenRouter to confirm traces appear in your backend.

## Compatible Platforms

Axiom, Jaeger, Grafana Tempo, Honeycomb, Lightstep, and self-hosted collectors.

## Custom Metadata Support

| Key | OTLP Mapping | Purpose |
|-----|--------------|---------|
| `trace_id` | Trace ID | Correlate multiple requests |
| `trace_name` | Span Name | Root span identifier |
| `span_name` | Span Name | Intermediate span naming |
| `generation_name` | Span Name | LLM generation identification |
| `parent_span_id` | Parent Span ID | Link to existing spans |

Custom metadata appears as span attributes in the `trace.metadata.*` namespace.

## Privacy Considerations

Privacy Mode excludes prompt and completion content while preserving token usage, costs, timing, and model information.
