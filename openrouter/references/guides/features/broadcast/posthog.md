# PostHog Broadcast Integration

## Overview

PostHog is an analytics platform enabling tracking and analysis of AI application usage. The integration allows automatic trace transmission from OpenRouter requests to PostHog's LLM analytics dashboard.

## Setup Instructions

### Step 1: Obtain API Credentials
Access PostHog project settings and locate your Project API Key (identifier begins with `phc_`).

### Step 2: Activate Broadcast Feature
Navigate to Settings > Observability in OpenRouter and enable the Broadcast toggle.

### Step 3: Configure PostHog Connection
Edit the PostHog configuration with:
- **Api Key**: Your project credentials
- **Endpoint** (optional): Use `https://us.i.posthog.com` (default) or `https://eu.i.posthog.com` for European regions

### Step 4: Validate Configuration
Test the connection; settings persist only upon successful verification.

### Step 5: Execute Test Request
Send an API call through OpenRouter and observe resulting analytics in PostHog.

## Custom Metadata Support

Attach contextual information via the `trace` field for enhanced analytics:

| Metadata Field | PostHog Property | Purpose |
|---|---|---|
| `trace_id` | Event property | Groups related events |
| `trace_name` | Event property | Identifies trace context |
| `generation_name` | Event property | Labels LLM generation events |

The `user` parameter maps to PostHog's user analytics property, while `session_id` enables session-level grouping.

## Privacy Considerations

When Privacy Mode is enabled, input and output choice properties are withheld. Token usage, costs, and model information remain available.
