# Broadcast to Arize AI

[Arize AX](https://arize.com) is an evaluation and observability platform developed by Arize AI; it offers tools for agent tracing, evals, prompt optimization, and more.

Connect Arize AI to automatically receive traces from your OpenRouter requests.

## Setup

### Step 1: Get your Arize credentials

Navigate to your space settings to find your API key and space key in Arize's Space Settings and API Keys sections. Identify your preferred Model ID for organizing traces.

### Step 2: Enable Broadcast in OpenRouter

Go to [Settings > Observability](https://openrouter.ai/settings/observability) and toggle **Enable Broadcast**.

### Step 3: Configure Arize AI

Click the edit icon next to **Arize AI** and enter:

- **Api Key**: Your Arize API key (required)
- **Space Key**: Your Arize space key (required)
- **Model Id**: Your model identifier for organizing traces (required)
- **Base Url** (optional): Defaults to `https://otlp.arize.com`

### Step 4: Test and save

Click **Test Connection** to verify the setup. The configuration only saves if the test passes.

### Step 5: Send a test trace

Make an API request through OpenRouter and view the trace in your Arize dashboard.

## Custom Metadata

OpenRouter uses OpenInference semantic conventions for Arize integration. Custom trace metadata maps to OTLP span attributes.

### Supported metadata keys

| Key | Arize Mapping | Description |
|-----|---------------|-------------|
| `trace_id` | Trace ID | Group multiple requests into a single trace |
| `trace_name` | Span Name | Custom name for the root trace |
| `span_name` | Span Name | Name for intermediate spans in the hierarchy |
| `generation_name` | Span Name | Name for the LLM generation span |
| `parent_span_id` | Parent Span ID | Link to an existing span in your trace hierarchy |

### Additional field mappings

- The `user` field maps to user identification in span attributes
- The `session_id` field maps to session tracking in span attributes
- Token usage, costs, and model parameters are automatically included as OpenInference-compatible attributes
- Custom metadata keys appear under the `metadata.*` namespace

### Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Classify this text..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "classification_pipeline_001",
    "trace_name": "Text Classification",
    "generation_name": "Classify Sentiment",
    "dataset": "customer_feedback",
    "experiment_id": "exp_v3"
  }
}
```

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. All other trace data -- token usage, costs, timing, model information, and custom metadata -- is still sent normally.
