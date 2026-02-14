# Uptime Optimization

## Overview

OpenRouter implements continuous monitoring of AI provider health to maximize application uptime. The platform tracks response times, error rates, and availability metrics in real-time to inform routing decisions.

## Core Functionality

The system operates by gathering performance data across all providers and using this information to make intelligent routing choices. This approach provides both reliability optimization and visibility into service performance.

As stated in the documentation: "OpenRouter continuously monitors the health and availability of AI providers to ensure maximum uptime for your applications."

## Available Examples

The page includes uptime tracking visualizations for:
- Claude 4 Sonnet (via Anthropic)
- Llama 3.3 70B Instruct (via Meta)

## Customization Options

Users can exercise control over provider selection through request parameters rather than relying solely on automatic routing. This flexibility maintains automatic fallback capabilities while allowing specific provider preferences.

## Related Resources

The documentation references additional guidance on [Provider Routing](/docs/features/provider-routing) for those seeking to customize their provider selection strategy.
