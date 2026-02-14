# Broadcast: Send Traces to External Observability Platforms

## Overview

OpenRouter's Broadcast feature enables automatic transmission of API request traces to external observability platforms. This functionality allows developers to monitor and analyze LLM usage across preferred tools without modifying application code.

## Enabling the Feature

Users can activate Broadcast through the Settings > Observability dashboard section, toggling the feature and adding destination endpoints. Organization administrators must authorize broadcast configuration changes for org-level accounts.

## Supported Destinations

The platform integrates with 16 stable destinations including:

- Arize AI, Braintrust, ClickHouse, Comet Opik, Datadog, Grafana Cloud, Langfuse, LangSmith, New Relic, OpenTelemetry Collector, PostHog, S3/S3-Compatible, Sentry, Snowflake, W&B Weave, and Webhook

Additional platforms in development include AWS Firehose, Dynatrace, Helicone, Portkey, and others.

## Trace Data Included

Standard traces contain request/response data, token usage metrics, costs, timing information, model details, and tool usage indicators. Multimodal content is excluded for efficiency.

Optional enrichment fields include:
- **User ID**: Associate traces with specific end-users (128 character limit)
- **Session ID**: Group related requests via `session_id` field or `x-session-id` header (128 character limit)
- **Custom Metadata**: Arbitrary JSON via the `trace` field

## Metadata Keys

Common metadata keys enable hierarchical trace organization:

| Key | Purpose |
|-----|---------|
| `trace_id` | Groups multiple API requests |
| `trace_name` | Custom root trace naming |
| `span_name` | Parent span creation |
| `generation_name` | Specific LLM call naming |
| `parent_span_id` | Links to external tracing systems |

## Configuration Options

**API Key Filtering**: Route traces from specific API keys to particular destinations, enabling isolated monitoring by use case.

**Sampling Rate**: Control trace transmission percentage (0.0-1.0) for cost management. Sampling is deterministic--sessions remain complete across destinations.

**Privacy Mode**: Excludes prompts and completions while preserving token counts, costs, timing, and metadata for compliance-focused implementations.

## Security & Performance

Destination credentials are encrypted at rest and decrypted only during trace transmission. Asynchronous processing prevents latency impact on API responses.
