# Datadog Broadcast Integration Guide

## Overview
"With Datadog LLM Observability, you can investigate the root cause of issues, monitor operational performance, and evaluate the quality, privacy, and safety of your LLM applications."

## Setup Instructions

### Step 1: Generate API Credentials
Navigate to Datadog's organization settings and create a new API key under the API Keys section.

### Step 2: Activate Broadcast
Visit OpenRouter's Settings > Observability page and enable the Broadcast feature.

### Step 3: Configure Connection
Select the Datadog option and provide:
- **Api Key**: Your credential from Datadog
- **Ml App**: Application identifier (e.g., "production-app")
- **Url** (optional): Regional endpoint; defaults to US5 region

### Step 4: Verify Setup
Use the connection test feature to confirm proper configuration before saving.

### Step 5: Validate Integration
Submit an API request through OpenRouter and confirm the trace appears in Datadog.

## Custom Metadata Support

The system supports organizing traces through several metadata keys:

| Key | Purpose |
|-----|---------|
| `trace_id` | Consolidate related requests |
| `trace_name` | Label root operation |
| `span_name` | Identify workflow steps |
| `generation_name` | Mark LLM interactions |

### Automatic Tags
- Service identifier based on configured app name
- User identification from request data

### Example Implementation
Custom fields in trace objects become searchable metadata within Datadog's interface.

## Privacy Features
When privacy mode is enabled, prompt and completion text is redacted while maintaining visibility into token usage, costs, timing, and model details.
