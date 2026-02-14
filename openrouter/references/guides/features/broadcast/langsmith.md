---
title: Broadcast to LangSmith
description: Send OpenRouter traces to LangSmith for LLM observability
---

# Broadcast to LangSmith

LangSmith is LangChain's platform for debugging, testing, evaluating, and monitoring LLM applications. With OpenRouter's Broadcast feature, traces are automatically sent to LangSmith without additional instrumentation.

## Setup

### Step 1: Get Your LangSmith Credentials

Obtain your API key from LangSmith's **Settings > API Keys** section. The key format is `lsv2_pt_...`. Also note your project name from your LangSmith workspace.

### Step 2: Enable Broadcast

Navigate to **Settings > Observability** in OpenRouter and activate the Broadcast feature.

### Step 3: Configure LangSmith Destination

Add LangSmith as a destination and provide:

- **Api Key**: Your LangSmith API key (format: `lsv2_pt_...`)
- **Project**: Your LangSmith project name
- **Endpoint** (optional): Defaults to `https://api.smith.langchain.com`. Update for self-hosted instances.

### Step 4: Test Connection

Click **Test Connection** to verify the setup. Configuration saves only upon successful validation.

### Step 5: Verify Traces

Send an API request through OpenRouter and confirm the trace appears in LangSmith. Traces include comprehensive data encompassing messages, token metrics, costs, model details, and performance measurements.

## Data Transmission

OpenRouter communicates with LangSmith via the OpenTelemetry (OTEL) protocol, including:

- **GenAI semantic conventions**: Model identifiers, token metrics, pricing details, and request parameters
- **LangSmith-specific attributes**: Trace identifiers, span classifications, and user information
- **Error events**: Exception classification and diagnostic messages when requests fail

## Custom Metadata

Enrich your traces with custom metadata by including the `trace` field in your API requests:

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Analyze this text..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_id": "analysis_workflow_123",
    "trace_name": "Text Analysis Pipeline",
    "span_name": "Sentiment Analysis",
    "generation_name": "Extract Sentiment",
    "environment": "production",
    "team": "nlp-team"
  }
}
```

### Metadata Mapping

| Key | LangSmith Mapping | Description |
|---|---|---|
| `trace_id` | Trace ID | Group multiple runs into a single trace |
| `trace_name` | Run Name | Custom name displayed in the LangSmith trace list |
| `span_name` | Run Name | Name for intermediate chain/tool runs |
| `generation_name` | Run Name | Name for the LLM run |
| `parent_span_id` | Parent Run ID | Link to an existing run in your trace hierarchy |

### Additional Context Fields

- The `user` field maps to LangSmith **User ID** for user tracking
- The `session_id` field maps to LangSmith **Session ID** for conversation tracking
- String arrays in metadata function as filterable tags
- Extra metadata keys within `trace` are available for filtering in LangSmith

### Run Type Mapping

- **GENERATION** maps to `llm` classification
- **SPAN** maps to `chain` classification
- **EVENT** maps to `tool` classification

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. Token usage, costs, timing, model information, and custom metadata remain transmitted.
