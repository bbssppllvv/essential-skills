# Grafana Cloud Documentation

## Overview
"Grafana Cloud is a fully-managed observability platform that includes Grafana Tempo for distributed tracing." OpenRouter transmits trace data via the OTLP HTTP/JSON protocol.

## Setup Process

### Step 1: Gather Credentials
You'll need three components from your Grafana Cloud portal:

1. **Base URL**: The OTLP endpoint (format: `https://otlp-gateway-prod-{region}.grafana.net`)
2. **Instance ID**: A numeric identifier for your Grafana Cloud instance
3. **API Key**: A token with write permissions (prefixed with `glc_`)

**Locating the OTLP endpoint:**
- Access your Grafana Cloud portal
- Navigate to Connections > Add new connection
- Find OpenTelemetry (OTLP) and select it
- Retrieve your endpoint URL

**Finding Instance ID:**
- Visit `https://grafana.com/orgs/{your-org}/stacks`
- Select your stack
- Locate the numeric ID in the URL or stack details

**Creating an API Token:**
- Go to My Account > Access Policies
- Establish a new policy with `traces:write` scope
- Generate and copy your token

### Step 2: Enable Broadcast
Access Settings > Observability and activate the Broadcast feature.

### Step 3: Configure Integration
Click the edit icon for Grafana Cloud and input your Base URL, Instance ID, and API Key.

### Step 4: Test Connection
Select "Test Connection" to validate your setup. Configuration saves only upon successful verification.

### Step 5: Verify with Test Trace
Make an API call through OpenRouter and confirm the trace appears in Grafana Cloud.

## Viewing Traces

### Method 1: TraceQL Exploration
1. Access your Grafana Cloud instance
2. Click Explore in the sidebar
3. Choose your Tempo data source
4. Switch to the TraceQL tab
5. Execute: `{ resource.service.name = "openrouter" }`

Filter by model: `{ resource.service.name = "openrouter" && span.gen_ai.request.model = "openai/gpt-4-turbo" }`

### Method 2: Drilldown Navigation
1. Go to your Grafana Cloud instance
2. Select Drilldown > Traces
3. Apply filters by service, duration, or attributes
4. Click any trace to view span details

## Trace Attributes

**Resource Level:**
- `service.name`: openrouter
- `service.version`: 1.0.0
- `openrouter.trace.id`: Unique trace identifier

**Span Level:**
- `gen_ai.operation.name`: Operation type (e.g., chat)
- `gen_ai.system`: Provider name
- `gen_ai.request.model`: Requested model
- `gen_ai.response.model`: Actual model used
- `gen_ai.usage.input_tokens`: Input token count
- `gen_ai.usage.output_tokens`: Output token count
- `gen_ai.response.finish_reason`: Completion reason

## Custom Metadata

Metadata from the `trace` field appears as span attributes under the `trace.metadata.*` namespace.

| Key | Grafana Mapping | Purpose |
|-----|-----------------|---------|
| `trace_id` | Trace ID | Group multiple requests |
| `trace_name` | Span Name | Root span name |
| `span_name` | Span Name | Intermediate span name |
| `generation_name` | Span Name | LLM generation span name |
| `parent_span_id` | Parent Span ID | Link to existing span |

**Example Request:**
```json
{
  "model": "openai/gpt-4o",
  "messages": [{"role": "user", "content": "Analyze this..."}],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "monitoring_001",
    "trace_name": "Analysis Pipeline",
    "generation_name": "Detection",
    "environment": "production"
  }
}
```

**Querying Custom Data:**
```traceql
{ resource.service.name = "openrouter" && span.trace.metadata.environment = "production" }
```

## Example Queries

**Slow Requests (>5 seconds):**
```traceql
{ resource.service.name = "openrouter" && duration > 5s }
```

**By User:**
```traceql
{ resource.service.name = "openrouter" && span.user.id = "user_abc123" }
```

**Errors:**
```traceql
{ resource.service.name = "openrouter" && status = error }
```

**By Model Pattern:**
```traceql
{ resource.service.name = "openrouter" && span.gen_ai.request.model =~ ".*gpt-4.*" }
```

## Troubleshooting

- **Missing traces**: Check time range, verify OTLP gateway URL (not main Grafana URL), confirm Instance ID is numeric and API key has write permissions, allow 1-2 minutes for propagation
- **Wrong data source**: Ensure you've selected the correct Tempo source, typically named `grafanacloud-{stack}-traces`

## Privacy Mode
When enabled, prompt and completion content is excluded from traces. Token usage, costs, timing, model information, and custom metadata remain visible.
