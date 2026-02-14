# Langfuse Broadcast Integration

## Overview

"Langfuse is an open-source LLM engineering platform for tracing, evaluating, and debugging LLM applications."

## Setup Instructions

### Step 1: Generate API Credentials
Navigate to Langfuse's Settings > API Keys section and generate a new key pair, noting both the Secret and Public keys.

### Step 2: Activate Broadcast Feature
Access OpenRouter's Settings > Observability page and enable the Broadcast functionality.

### Step 3: Configure Connection Details
Edit the Langfuse integration by inputting:
- Secret Key from Langfuse
- Public Key from Langfuse
- Base URL (defaults to `https://us.cloud.langfuse.com`, customizable for other regions)

### Step 4: Validate Configuration
Use the Test Connection button to confirm proper setup before saving.

### Step 5: Verify with Test Request
Send an API call through OpenRouter and confirm the trace appears in Langfuse.

## Custom Metadata Options

| Parameter | Langfuse Field | Purpose |
|-----------|---|---|
| `trace_id` | Trace ID | Consolidate multiple requests |
| `trace_name` | Trace Name | Display label in Langfuse |
| `span_name` | Span Name | Intermediate hierarchy naming |
| `generation_name` | Generation Name | LLM operation identifier |
| `parent_span_id` | Parent Observation ID | Establish span relationships |

Additional fields like `user` and `session_id` map to corresponding Langfuse analytics dimensions.

## Privacy Considerations

When Privacy Mode is enabled, request content is withheld while metrics, timing, costs, and metadata continue transmitting normally.
