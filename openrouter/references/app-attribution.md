# App Attribution Documentation

## Overview

App attribution enables developers to connect their API usage with their applications on OpenRouter. By including two simple HTTP headers in requests, apps gain visibility in public rankings and access to detailed analytics.

## Key Benefits

According to the documentation, attributed apps receive:

- **Public visibility** through daily, weekly, and monthly leaderboards
- **Model-specific features** where apps appear on individual model pages
- **Comprehensive analytics** showing token consumption and usage trends
- **Developer recognition** within the OpenRouter community

## Attribution Implementation

The system uses two optional headers:

**HTTP-Referer**: "identifies your app's URL and is used as the primary identifier for rankings"

**X-Title**: "sets or modifies your app's display name in rankings and analytics"

The documentation notes that "apps using localhost URLs must include a title to be tracked."

## Code Examples

Implementation is straightforward across multiple languages:

- **TypeScript SDK**: Use `defaultHeaders` object with both headers
- **Python (OpenAI SDK)**: Pass headers via `extra_headers` parameter
- **Direct API calls**: Include headers in the request
- **cURL**: Use `-H` flags for each header

## Where Apps Appear

Attributed applications display in:

1. **Main rankings** showing top apps by token usage
2. **Model-specific tabs** listing which apps use particular models most
3. **Individual analytics pages** at `openrouter.ai/apps?url=<app-url>`

## Best Practices

- Use primary domain URLs (avoid subdomains for single applications)
- Create descriptive titles reflecting the actual app name
- Include a title when developing locally
