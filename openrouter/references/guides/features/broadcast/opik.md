# Broadcast to Comet Opik

[Comet Opik](https://www.comet.com/site/products/opik/) is an open-source platform for evaluating, testing, and monitoring LLM applications.

Connect Comet Opik to automatically receive traces from your OpenRouter requests.

## Setup

### Step 1: Get your Opik credentials

Log in to Comet, create a workspace and project, then retrieve your API key from Settings > API Keys. The key starts with `opik_...`.

### Step 2: Enable Broadcast in OpenRouter

Go to [Settings > Observability](https://openrouter.ai/settings/observability) and toggle **Enable Broadcast**.

### Step 3: Configure Comet Opik

Click the edit icon next to **Comet Opik** and enter:

- **API Key**: Your Opik API key (starts with `opik_...`)
- **Workspace**: Your Comet workspace name
- **Project**: The project where traces will be stored

### Step 4: Test and save

Click **Test Connection** to verify the setup. The configuration only saves if the test passes.

### Step 5: Send a test trace

Make an API request through OpenRouter and view the trace in the Opik dashboard.

## Custom Metadata

Opik allows custom metadata on traces and spans for organization and filtering.

### Supported metadata keys

| Key | Opik Mapping | Description |
|-----|--------------|-------------|
| `trace_id` | Trace metadata (`openrouter_trace_id`) | Group multiple requests into a single trace |
| `trace_name` | Trace Name | Custom name displayed in the Opik trace list |
| `span_name` | Span Name | Name for intermediate spans in the hierarchy |
| `generation_name` | Span Name | Name for the LLM generation span |

### Additional field mappings

- The `user` field maps to user identification in trace metadata
- OpenRouter IDs are stored as `openrouter_trace_id` and `openrouter_observation_id`
- Custom metadata keys from the `trace` object are included in both trace and span metadata

### Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Evaluate this response..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_name": "Response Quality Eval",
    "generation_name": "Quality Assessment",
    "eval_suite": "quality_v2",
    "test_case_id": "tc_001"
  }
}
```

## Data Transmitted

The following data is automatically included in spans:

- Cost information (input, output, total costs)
- Model parameters (when available)
- Finish reasons (when available)
- Token usage metrics

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. All other trace data -- token usage, costs, timing, model information, and custom metadata -- is still sent normally.
