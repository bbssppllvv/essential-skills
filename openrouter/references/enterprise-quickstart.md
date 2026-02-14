# Enterprise Quickstart - OpenRouter Documentation

## Organization Setup
Teams can collaborate using shared credits and centralized API key management. Users navigate to Settings > Preferences to create organizations, then invite members and manage roles (Admin and Member). Key features include "shared credit pools for centralized billing" and organization-wide activity tracking.

## API Key Management
The Management API enables automated provisioning and lifecycle management of credentials. Features include programmatic key creation for customer instances and automated rotation supporting zero-downtime transitions. For those using BYOK, "you can rotate OpenRouter API keys without touching your provider credentials, simplifying key management."

## Security Implementation
**Guardrails** allow organizations to set spending limits (daily, weekly, or monthly), restrict model/provider access, and enforce Zero Data Retention. These controls apply to members or specific API keys, with stricter rules taking precedence when multiple guardrails exist.

**Data Protection**: OpenRouter doesn't retain prompts or responses unless explicitly opted into prompt logging. "Only metadata (token counts, latency, etc.) is stored for reporting and your activity feed."

## Observability Configuration
**Broadcast** automatically sends traces to external platforms (Datadog, Langfuse, LangSmith, etc.) without additional instrumentation. Users can filter by API key and set sampling rates at Settings > Observability.

**User Tracking** involves including a `user` parameter in API requests to improve caching and enable user-level analytics.

## Usage Monitoring & Cost Tracking
API responses include detailed usage data: token counts, costs in credits, and timing information. Users can export aggregated data as CSV or PDF from the Activity page, filtering by time period and grouping options.

## Reliability Optimization
The platform monitors provider health and automatically routes around outages. Users can configure fallback chains using multiple models and customize provider selection based on cost, latency, or preferences.
