# Comet Opik Integration Guide

## Overview
Comet Opik is "an open-source platform for evaluating, testing, and monitoring LLM applications." This documentation explains how to connect it with OpenRouter to automatically capture and analyze API request traces.

## Setup Steps

### Step 1: Obtain Opik Credentials
Users must access their Comet account to establish an Opik workspace and project, then navigate to API settings to generate or retrieve their authentication key.

### Step 2: Activate Broadcast Feature
Navigate to the Observability settings section in OpenRouter and enable the broadcast functionality.

### Step 3: Configure Integration
Click the edit option for Comet Opik and provide three pieces of information:
- **Api Key**: Authentication credential (formatted as `opik_...`)
- **Workspace**: Comet workspace identifier
- **Project Name**: Destination project for trace logging

### Step 4: Verify Configuration
Test the connection to ensure all settings are correct; the system saves configuration only upon successful validation.

### Step 5: Validate with Test Request
Send a request through OpenRouter and confirm the trace appears in the Opik dashboard.

## Custom Metadata Support

Comet Opik allows custom metadata on traces and spans. Supported fields include:

| Field | Maps To | Purpose |
|-------|---------|---------|
| `trace_id` | Trace metadata | Groups multiple requests |
| `trace_name` | Trace Name | Custom display label |
| `span_name` | Span Name | Intermediate span identifier |
| `generation_name` | Span Name | LLM generation identifier |

### Metadata Example
Custom metadata passes through request objects and gets attached to trace and span data. The system automatically includes cost metrics, model parameters, and finish reasons.

## Privacy Considerations
When privacy mode is enabled, "prompt and completion content is excluded from traces." Other data--usage statistics, costs, timing, and model details--remains transmitted normally.
