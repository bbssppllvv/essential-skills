# Latency and Performance

Learn about OpenRouter's performance characteristics, latency optimizations, and best practices for achieving optimal response times.

## Overview

OpenRouter prioritizes performance by minimizing gateway latency. The platform achieves minimal overhead through:

- **Edge computing** using Cloudflare Workers to stay as close as possible to your application
- **Efficient caching** of user and API key data at the edge
- **Optimized routing logic** for fast request processing

## Performance Considerations

### Cache Warming

During initial operation in a new region (first 1-2 minutes), users may encounter slightly elevated latency as caches populate. This normalizes quickly after the initial warm-up period.

### Credit Balance Checks

OpenRouter performs additional database checks when:

- A user's credit balance is low (single digit dollars)
- An API key approaches its configured limit

This more aggressive cache expiration increases latency temporarily until additional credits are added to the account.

### Model Fallback

When using model or provider routing, failed primary options trigger automatic failover attempts. This adds latency to that specific request. However, the system intelligently tracks failures to avoid repeated latency penalties by routing around unavailable providers in subsequent requests.

## Best Practices

### Maintain a Healthy Credit Balance

Set up auto-topup with a higher threshold and amount. A recommended minimum balance of $10-20 helps avoid forced credit checks that degrade response times. This prevents zero-balance scenarios that trigger additional database verification on every request.

### Use Provider Preferences

Leverage provider routing features to optimize for your specific latency requirements. You can target:

- **Time-to-first-token** — Prioritize providers that begin streaming responses quickly
- **Overall completion speed** — Optimize for total response time while managing costs effectively
