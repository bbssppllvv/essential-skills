# Braintrust Broadcast Integration

## Overview

"Braintrust is an end-to-end platform for evaluating, monitoring, and improving LLM applications." This integration enables automatic trace transmission from OpenRouter requests to Braintrust.

## Setup Instructions

### Step 1: Obtain Credentials
Access Braintrust's Account Settings to generate an API key and locate your Project ID within project settings.

### Step 2: Enable Broadcast Feature
Navigate to OpenRouter's Settings > Observability and activate the Broadcast toggle.

### Step 3: Configure Integration
Select the edit option for Braintrust and provide:
- **Api Key**: Your Braintrust authentication credential
- **Project Id**: Your Braintrust project identifier
- **Base Url** (optional): Defaults to `https://api.braintrust.dev`

### Step 4: Validate Connection
Click "Test Connection" to confirm setup validity. Configuration persists only upon successful testing.

### Step 5: Verify with Test Trace
Execute an OpenRouter API request and confirm trace visibility in Braintrust.

## Custom Metadata Support

Braintrust accepts custom metadata, tags, and hierarchical span structures for organizing logs.

| Key | Braintrust Mapping | Purpose |
|-----|-------------------|---------|
| `trace_id` | Span/Root Span ID | Consolidate multiple logs into single trace |
| `trace_name` | Name | Custom display name in log view |
| `span_name` | Name | Intermediate span hierarchy identifier |
| `generation_name` | Name | LLM span designation |

## Data Transmission

Braintrust receives comprehensive metrics including token counts, cached token usage, reasoning tokens, cost breakdowns, and timing data. User and session identifiers map to corresponding Braintrust metadata fields.

## Privacy Considerations

When Privacy Mode is enabled, prompt and completion text excludes from traces while preserving token usage, costs, timing, model details, and custom metadata.
