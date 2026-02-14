# API Authentication Documentation

## Overview

OpenRouter uses Bearer token authentication for API requests, compatible with `curl` and the OpenAI SDK. The platform's API keys are notably more powerful than direct model API keys, offering credit limit controls and OAuth integration capabilities.

## Key Authentication Method

Users must first generate an API key through OpenRouter's dashboard, optionally setting spending limits. When calling the API directly, include the `Authorization` header with your Bearer token.

## Implementation Examples

**TypeScript with OpenRouter SDK:** Configure with your API key and optional site headers for ranking attribution.

**Python with OpenAI SDK:** Set the base URL to `https://openrouter.ai/api/v1` and provide your API key.

**Raw API calls:** Use fetch or cURL with the authorization header formatted as "Bearer [YOUR_KEY]".

Optional headers include `HTTP-Referer` and `X-Title` for site attribution purposes.

## Security Considerations

**Critical reminder:** You must protect your API keys and never commit them to public repositories. OpenRouter monitors for exposed keys as a GitHub secret scanning partner and sends notifications if compromise is detected.

Immediate action required upon exposure: delete the compromised key and generate a replacement via the settings page. Environment variables are strongly recommended for key storage.
