# OpenRouter Quickstart Guide

OpenRouter provides a unified API for accessing hundreds of AI models through a single endpoint, with automatic fallback handling and cost-effective model selection.

## Getting Started Options

The platform supports three primary integration methods:

### 1. OpenRouter SDK (Beta)

Installation via package managers:
```bash
npm install @openrouter/sdk
yarn add @openrouter/sdk
pnpm add @openrouter/sdk
```

Basic usage involves initializing the SDK with your API key and optional site attribution headers, then making chat completion requests.

### 2. Direct API Integration

The service accepts HTTP POST requests to `https://openrouter.ai/api/v1/chat/completions` with:
- Bearer token authentication
- Optional referrer and title headers for leaderboard attribution
- JSON payload containing model selection and message arrays

Supported implementation languages include Python, TypeScript (fetch), and shell/curl commands.

### 3. OpenAI SDK Compatibility

The OpenAI Python and TypeScript libraries work with OpenRouter by configuring the base URL to `https://openrouter.ai/api/v1` and providing your API credentials.

## Key Features

- Optional HTTP-Referer and X-Title headers enable app attribution and leaderboard rankings
- Streaming support available across all integration methods
- Interactive Request Builder tool generates code in multiple languages
- Additional documentation covers frameworks, rate limits, and community integrations
