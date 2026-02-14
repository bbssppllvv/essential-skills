# BYOK Documentation - OpenRouter

## Overview

OpenRouter's Bring Your Own Keys (BYOK) feature enables users to leverage their existing AI provider API credentials while utilizing OpenRouter's unified interface. "Your provider keys are securely encrypted and used for all requests routed through the specified provider."

## Key Management & Costs

Users can manage their provider keys through account settings. The platform charges "a small fee based on usage" for BYOK services, with this fee waived for the first monthly threshold of requests.

## Routing Behavior

### Default Priority System

OpenRouter prioritizes user-provided keys whenever available. If a key encounters rate limiting or failures, the system automatically falls back to shared OpenRouter credits by default.

### The "Always Use This Key" Option

Users can configure individual keys with an enforcement setting to prevent fallback to shared credits, ensuring all requests use their account exclusively—though this may trigger rate limit errors if capacity is exhausted.

### Integration with Provider Ordering

When combining BYOK with provider ordering, "OpenRouter always prioritizes BYOK endpoints first, regardless of where that provider appears in your specified order." After exhausting BYOK options, the system falls back to shared capacity following the user's specified sequence.

**Example routing hierarchy with three BYOK providers:**
- All BYOK endpoints (in provider order)
- Shared OpenRouter capacity (in provider order)

### Multiple Keys for Same Provider

The platform supports multiple BYOK keys per provider, though "the order in which multiple keys for the same provider are tried is not guaranteed."

## Provider-Specific Configurations

### Azure AI Services

Azure integration requires JSON configuration with:
- Model slug (OpenRouter identifier)
- Endpoint URL (with `/chat/completions` suffix)
- API key
- Azure model deployment ID

### AWS Bedrock

Two authentication methods available:

**Option 1: Bedrock API Keys** - Simple string format, tied to specific regions

**Option 2: AWS Credentials** - JSON format with access key ID, secret key, and region specification

Required IAM permissions include `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream`.

### Google Vertex AI

Authentication uses a Google Cloud service account JSON key with optional region specification (defaults to global).

Required permissions: `aiplatform.endpoints.predict` and `aiplatform.endpoints.streamingPredict`.

## Troubleshooting

Users can debug issues via the Activity page's raw metadata viewer, which displays provider HTTP response codes.

Common error codes:
- **400**: Invalid request format
- **401**: Invalid or revoked API key
- **403**: Insufficient permissions
- **429**: Rate limit exceeded
- **500**: Provider-side server error
