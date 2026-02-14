# Arize AI Integration Guide

## Overview

"Arize AX is an evaluation and observability platform developed by Arize AI; it offers tools for agent tracing, evals, prompt optimization, and more."

## Setup Instructions

### Step 1: Obtain Arize Credentials
Access your Arize account and retrieve:
- Space Key (via Space Settings)
- API Key (via API Keys section)
- Model ID for trace organization

### Step 2: Enable Broadcast in OpenRouter
Navigate to Settings > Observability and activate the broadcast feature.

### Step 3: Configure Arize Integration
Provide these details when editing the Arize AI configuration:
- **Api Key**: Your authentication credential
- **Space Key**: Your workspace identifier
- **Model Id**: Identifier for organizing traces
- **Base Url** (optional): Defaults to `https://otlp.arize.com`

### Step 4: Validate Connection
Use the "Test Connection" button to verify setup before saving.

### Step 5: Send Test Trace
Execute an API request through OpenRouter and inspect the resulting trace in your Arize dashboard.

## Custom Metadata Support

The platform follows the OpenInference semantic convention. Custom metadata from the `trace` field becomes span attributes in the OTLP payload.

### Available Metadata Keys

| Key | Arize Mapping | Purpose |
|-----|---------------|---------|
| `trace_id` | Trace ID | Group multiple requests |
| `trace_name` | Span Name | Custom root trace name |
| `span_name` | Span Name | Intermediate span naming |
| `generation_name` | Span Name | LLM generation span naming |
| `parent_span_id` | Parent Span ID | Span hierarchy linking |

### Example Request
Additional fields like `user` and `session_id` map to span attributes automatically, while token usage and costs are included as OpenInference-compatible attributes.

## Privacy Considerations

When Privacy Mode is enabled, prompt and completion content is excluded from traces, though all other data remains transmitted.
