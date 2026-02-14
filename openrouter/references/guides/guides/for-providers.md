# Provider Integration Documentation

## Overview

OpenRouter enables AI model providers to integrate their inference services through a unified API. Interested providers should complete the registration form to begin the onboarding process.

## Core Requirements

### 1. Models List Endpoint

Providers must create an endpoint that returns all available models in a standardized JSON format. The response should include:

**Required Fields:**
- Model identifier (`id`)
- Human-readable name
- Creation timestamp
- Input/output modalities
- Context and output length limits
- Pricing structure (in USD)

**Optional Fields:**
- Model description
- Deprecation date
- Datacenter location information
- Feature support flags

**Pricing Structure:**

Pricing uses string format to prevent floating-point errors. Costs are specified per token or per request. The system supports tiered pricing for models with variable costs based on "context length thresholds, allowing up to two distinct pricing tiers."

**Supported Parameters:**

Models can declare compatibility with sampling parameters including temperature, top_p, top_k, and seed. Feature support includes tools integration, JSON mode, and reasoning capabilities.

### 2. Payment Integration

OpenRouter requires automated payment processing through either auto top-up or invoicing mechanisms to enable service usage.

### 3. Uptime Monitoring

The platform continuously tracks provider reliability, calculating uptime as "successful requests / total requests." Authentication failures, server errors, and mid-stream failures impact metrics. Traffic automatically routes to higher-performing providers, with degraded status applied below 95% uptime.

### 4. Performance Tracking

OpenRouter publicly measures time-to-first-token (TTFT) and throughput metrics. Providers should minimize queueing and maintain consistent streaming to optimize competitive positioning on the platform.
