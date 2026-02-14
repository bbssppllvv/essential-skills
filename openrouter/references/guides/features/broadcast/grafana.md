# Broadcast to Grafana Cloud

[Grafana Cloud](https://grafana.com/products/cloud/) is a fully-managed observability platform that includes Grafana Tempo for distributed tracing. OpenRouter sends traces via the standard OTLP HTTP/JSON endpoint.

## Setup

### Step 1: Get your Grafana Cloud credentials

You need three values from your Grafana Cloud portal:

1. **Base URL**: Your Grafana Cloud OTLP endpoint (e.g., `https://otlp-gateway-prod-us-west-0.grafana.net`)
2. **Instance ID**: Your numeric Grafana Cloud instance ID (e.g., `123456`)
3. **API Key**: A Grafana Cloud API token with write permissions (starts with `glc_...`)

**Finding your OTLP endpoint:**

- Log in to your Grafana Cloud portal
- Navigate to **Connections** > **Add new connection**
- Search for **OpenTelemetry (OTLP)** and select it
- On the configuration page, you'll find your **OTLP endpoint URL**

> **Tip:** The base URL should be the OTLP gateway endpoint, not your main Grafana dashboard URL. The format is `https://otlp-gateway-prod-{region}.grafana.net`.

**Finding your Instance ID:**

- Go to your Grafana Cloud account at `https://grafana.com/orgs/{your-org}/stacks`
- Select your stack
- Your **Instance ID** is the numeric value shown in the URL or on the stack details page

**Creating an API token:**

- In Grafana Cloud, go to **My Account** > **Access Policies**
- Create a new access policy with `traces:write` scope
- Generate a token from this policy
- Copy the token (starts with `glc_...`)

### Step 2: Enable Broadcast in OpenRouter

Go to [Settings > Observability](https://openrouter.ai/settings/observability) and toggle **Enable Broadcast**.

### Step 3: Configure Grafana Cloud

Click the edit icon next to **Grafana Cloud** and enter:

- **Base URL**: Your Grafana Cloud OTLP endpoint (e.g., `https://otlp-gateway-prod-us-west-0.grafana.net`)
- **Instance ID**: Your numeric Grafana Cloud instance ID
- **API Key**: Your Grafana Cloud API token with write permissions

### Step 4: Test and save

Click **Test Connection** to verify the setup. The configuration only saves if the test passes.

### Step 5: Send a test trace

Make an API request through OpenRouter and view the trace in Grafana Cloud.

> **Note:** There can be a 1-2 minute delay before traces appear in Grafana.

## Viewing Traces

### Method 1: TraceQL Exploration

1. Access your Grafana Cloud instance
2. Click **Explore** in the sidebar
3. Choose your Tempo data source
4. Switch to the **TraceQL** tab
5. Execute: `{ resource.service.name = "openrouter" }`

### Method 2: Drilldown Navigation

1. Go to your Grafana Cloud instance
2. Select **Drilldown** > **Traces**
3. Apply filters by service, duration, or attributes
4. Click any trace to view span details

## Querying Traces

OpenRouter traces include the following key attributes:

### Resource Attributes

- `service.name`: Always `openrouter`
- `service.version`: `1.0.0`
- `openrouter.trace.id`: The OpenRouter trace ID

### Span Attributes

- `gen_ai.operation.name`: The operation type (e.g., `chat`)
- `gen_ai.system`: The AI provider (e.g., `openai`)
- `gen_ai.request.model`: The requested model
- `gen_ai.response.model`: The actual model used
- `gen_ai.usage.input_tokens`: Number of input tokens
- `gen_ai.usage.output_tokens`: Number of output tokens
- `gen_ai.usage.total_tokens`: Total tokens used
- `gen_ai.response.finish_reason`: Why the generation ended (e.g., `stop`)

### TraceQL Examples

**View all OpenRouter traces:**

```traceql
{ resource.service.name = "openrouter" }
```

**Filter by specific model:**

```traceql
{ resource.service.name = "openrouter" && span.gen_ai.request.model = "openai/gpt-4-turbo" }
```

**Find slow requests (> 5 seconds):**

```traceql
{ resource.service.name = "openrouter" && duration > 5s }
```

**Find requests by user:**

```traceql
{ resource.service.name = "openrouter" && span.user.id = "user_abc123" }
```

**Find errors:**

```traceql
{ resource.service.name = "openrouter" && status = error }
```

**Find requests by model (pattern matching):**

```traceql
{ resource.service.name = "openrouter" && span.gen_ai.request.model =~ ".*gpt-4.*" }
```

**Query custom metadata by environment:**

```traceql
{ resource.service.name = "openrouter" && span.trace.metadata.environment = "production" }
```

**Query custom metadata by alert ID:**

```traceql
{ resource.service.name = "openrouter" && span.trace.metadata.alert_id = "alert_789" }
```

## Custom Metadata

OpenRouter supports custom metadata on traces for organizing and filtering your LLM telemetry in Grafana.

### Supported metadata keys

| Key | Grafana Mapping | Description |
|-----|-----------------|-------------|
| `trace_id` | Trace ID | Group multiple requests into a single trace |
| `trace_name` | Span Name | Custom name for the root span |
| `span_name` | Span Name | Name for intermediate spans in the hierarchy |
| `generation_name` | Span Name | Name for the LLM generation span |
| `parent_span_id` | Parent Span ID | Link to an existing span in your trace hierarchy |

### Additional field mappings

- The `user` field maps to `user.id` in span attributes
- The `session_id` field maps to `session.id` in span attributes
- Custom metadata keys from `trace` appear under the `trace.metadata.*` namespace in span attributes

### Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Analyze this metric..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "monitoring_pipeline_001",
    "trace_name": "Metric Analysis Pipeline",
    "generation_name": "Anomaly Detection",
    "environment": "production",
    "alert_id": "alert_789"
  }
}
```

## Troubleshooting

- **Missing traces**: Check time range, verify OTLP gateway URL (not main Grafana URL), confirm Instance ID is numeric and API key has write permissions, allow 1-2 minutes for propagation
- **Wrong data source**: Ensure you've selected the correct Tempo source, typically named `grafanacloud-{stack}-traces`

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. All other trace data -- token usage, costs, timing, model information, and custom metadata -- is still sent normally.
