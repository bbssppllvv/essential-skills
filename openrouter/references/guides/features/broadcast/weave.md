# W&B Weave Broadcast Integration

## Overview

"Weights & Biases Weave is an observability platform for tracking and evaluating LLM applications." This integration enables automatic trace delivery from OpenRouter requests to your W&B workspace.

## Setup Instructions

### Step 1: Obtain W&B Credentials
Navigate to your W&B User Settings and retrieve your API key for authentication purposes.

### Step 2: Activate Broadcast Feature
Access OpenRouter's Settings > Observability section and enable the Broadcast toggle.

### Step 3: Configure W&B Weave Settings
Click the edit option for W&B Weave and provide:
- **Api Key**: Your W&B authentication credential
- **Entity**: Username or team identifier
- **Project**: Target project name for trace logging
- **Base Url** (optional): Defaults to `https://trace.wandb.ai`

### Step 4: Verify Configuration
"Click Test Connection to verify the setup. The configuration only saves if the test passes."

### Step 5: Generate a Test Trace
Execute an API request via OpenRouter and inspect the resulting trace in W&B Weave.

## Custom Metadata Support

You can enrich traces with structured data:

| Key | Weave Mapping | Purpose |
|-----|---------------|---------|
| `trace_id` | `openrouter_trace_id` attribute | Custom identifier |
| `trace_name` | `op_name` | Operation display name |
| `generation_name` | `op_name` | LLM call identifier |

### Example Payload
```json
{
  "model": "openai/gpt-4o",
  "messages": [{"role": "user", "content": "Write a poem about AI..."}],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_name": "Creative Writing Agent",
    "prompt_template": "poem_v2"
  }
}
```

## Data Organization

Traces organize into three categories: attributes (metadata), inputs (request data and parameters), and summary (token usage, costs, timing).

## Privacy Considerations

When Privacy Mode is enabled, prompt and completion text is excluded, while metrics and metadata remain transmitted.
