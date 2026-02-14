# LangSmith Integration Guide

## Overview

LangSmith is "LangChain's platform for debugging, testing, evaluating, and monitoring LLM applications." This integration enables automatic trace transmission from OpenRouter to LangSmith.

## Setup Instructions

### Step 1: Obtain Credentials
Access LangSmith's **Settings > API Keys** to generate an API key, then locate your project name or establish a new project.

### Step 2: Enable Broadcast Feature
Navigate to OpenRouter's **Settings > Observability** and activate the Broadcast toggle.

### Step 3: Configure LangSmith Connection
Select the edit option adjacent to LangSmith and input:
- **Api Key**: Your LangSmith API key (format: `lsv2_pt_...`)
- **Project**: Your selected project name
- **Endpoint** (optional): Defaults to `https://api.smith.langchain.com` for self-hosted alternatives

### Step 4: Verify Configuration
Execute the **Test Connection** feature to confirm proper setup. Configuration persists only upon successful testing.

### Step 5: Transmit Test Trace
Submit an API request via OpenRouter and examine the resulting trace in LangSmith. Traces include comprehensive data encompassing messages, token metrics, expenses, model details, and performance measurements.

## Data Transmission

OpenRouter utilizes the OpenTelemetry (OTEL) protocol to transmit traces featuring:
- GenAI semantic conventions (model identifiers, token metrics, expenses)
- LangSmith-specific attributes (trace naming, span classification, user identifiers)
- Error documentation with exception details when failures occur

## Custom Metadata Framework

### Supported Metadata Mapping

| Metadata Key      | LangSmith Destination | Purpose                          |
|-------------------|----------------------|----------------------------------|
| `trace_id`        | Trace ID             | Consolidate runs into single trace |
| `trace_name`      | Run Name             | Display name in trace inventory  |
| `span_name`       | Run Name             | Intermediate operation naming    |
| `generation_name` | Run Name             | LLM operation identifier        |
| `parent_span_id`  | Parent Run ID        | Hierarchical trace linking      |

String arrays in metadata function as filterable tags. The `user` field corresponds to LangSmith's User ID, while `session_id` enables conversation monitoring.

### Run Type Mapping
- **GENERATION** → `llm` classification
- **SPAN** → `chain` classification
- **EVENT** → `tool` classification

## Privacy Considerations

Enabling Privacy Mode excludes prompt and completion text from traces while maintaining transmission of token usage, expenses, timing information, model specifications, and custom attributes.
