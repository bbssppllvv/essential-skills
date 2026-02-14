# Broadcast to Braintrust

[Braintrust](https://www.braintrust.dev) is an end-to-end platform for evaluating, monitoring, and improving LLM applications.

Connect Braintrust to automatically receive traces from your OpenRouter requests.

## Setup

### Step 1: Get your Braintrust API key and Project ID

Navigate to Braintrust's Account Settings to create an API key. Locate your Project ID in your project settings.

### Step 2: Enable Broadcast in OpenRouter

Go to [Settings > Observability](https://openrouter.ai/settings/observability) and toggle **Enable Broadcast**.

### Step 3: Configure Braintrust

Click the edit icon next to **Braintrust** and enter:

- **Api Key**: Your Braintrust API key
- **Project Id**: Your Braintrust project ID
- **Base Url** (optional): Defaults to `https://api.braintrust.dev`

### Step 4: Test and save

Click **Test Connection** to verify the setup. The configuration only saves if the test passes.

### Step 5: Send a test trace

Make an API request through OpenRouter and view the trace in Braintrust.

## Custom Metadata

Braintrust supports custom metadata, tags, and nested span structures for organizing your LLM logs.

### Supported metadata keys

| Key | Braintrust Mapping | Description |
|-----|-------------------|-------------|
| `trace_id` | Span ID / Root Span ID | Group multiple logs into a single trace |
| `trace_name` | Name | Custom name displayed in the Braintrust log view |
| `span_name` | Name | Name for intermediate spans in the hierarchy |
| `generation_name` | Name | Name for the LLM span |

### Additional field mappings

- The `user` field maps to Braintrust's `user_id` in metadata
- The `session_id` field maps to `session_id` in metadata
- Custom metadata keys are included in the span's metadata object
- Tags are passed through for filtering in the Braintrust UI

### Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Generate a summary..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "eval_run_456",
    "trace_name": "Summarization Eval",
    "generation_name": "GPT-4o Summary",
    "eval_dataset": "news_articles",
    "experiment_id": "exp_789"
  }
}
```

## Data Transmitted

Braintrust receives comprehensive telemetry data including:

- Token counts (prompt, completion, total)
- Cached token usage when available
- Reasoning token counts for supported models
- Cost information (input, output, total costs)
- Duration and timing metrics

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. All other trace data -- token usage, costs, timing, model information, and custom metadata -- is still sent normally.
