# Latency and Performance

## Overview

OpenRouter prioritizes performance by minimizing gateway latency through "edge computing using Cloudflare Workers to stay as close as possible to your application" and "efficient caching of user and API key data at the edge."

## Key Performance Factors

**Cache Warming**
During initial deployment in new regions, expect temporarily elevated latency as edge caches populate over the first 1-2 minutes.

**Credit Balance Monitoring**
OpenRouter implements additional database checks when account balances are critically low or API keys approach spending limits. This protective measure "increases latency until additional credits are added" to prevent billing overages.

**Model and Provider Fallback**
When primary routing fails, automatic failover mechanisms engage. The system learns from failures to intelligently route around unavailable providers, reducing repeated latency penalties.

## Optimization Recommendations

**Maintain Adequate Funds**
Keep account balances between $10-20 by configuring automatic top-ups. This eliminates forced credit verification checks that degrade response times.

**Leverage Provider Routing**
Use provider preferences to align performance with your specific needs—whether optimizing time-to-first-token or overall completion speed while managing costs effectively.
