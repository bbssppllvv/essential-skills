# Broadcast to New Relic

[New Relic](https://newrelic.com) is a full-stack observability platform for monitoring applications, infrastructure, and digital experiences.

Connect New Relic to automatically receive traces from your OpenRouter requests.

## Setup

### Step 1: Get your New Relic license key

Navigate to your New Relic account's API Keys section and retrieve an **Ingest - License** key.

### Step 2: Enable Broadcast in OpenRouter

Go to [Settings > Observability](https://openrouter.ai/settings/observability) and toggle **Enable Broadcast**.

### Step 3: Configure New Relic

Click the edit icon next to **New Relic** and enter:

- **License Key**: Your New Relic Ingest - License key
- **Region**: Select your region (`us` or `eu`)

### Step 4: Test and save

Click **Test Connection** to verify the setup. The configuration only saves if the test passes.

### Step 5: Send a test trace

Make an API request through OpenRouter and view the trace in New Relic's distributed tracing view.

## Custom Metadata

OpenRouter supports custom metadata on traces for organizing and filtering your LLM telemetry in New Relic. Traces reach New Relic through the OTLP protocol.

### Supported metadata keys

| Key | New Relic Mapping | Description |
|-----|-------------------|-------------|
| `trace_id` | Trace ID | Group multiple requests into a single trace |
| `trace_name` | Span Name | Custom name for the root span |
| `span_name` | Span Name | Name for intermediate spans in the hierarchy |
| `generation_name` | Span Name | Name for the LLM generation span |
| `parent_span_id` | Parent Span ID | Link to an existing span in your trace hierarchy |

### Additional field mappings

- The `user` field maps to `user.id` in span attributes
- The `session_id` field maps to `session.id` in span attributes
- Custom metadata keys from `trace` are included as span attributes under the `trace.metadata.*` namespace
- GenAI semantic conventions (`gen_ai.*` attributes) are used for model, token, and cost data

### Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Summarize this report..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "workflow_789",
    "trace_name": "Report Processing",
    "generation_name": "Summarize Report",
    "environment": "production",
    "service": "report-api"
  }
}
```

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. All other trace data -- token usage, costs, timing, model information, and custom metadata -- is still sent normally.
