# DevTools Documentation

## Overview

OpenRouter's DevTools provides telemetry capture and visualization capabilities for SDK development. DevTools is designed for development use only and should never be deployed in production environments.

## Core Components

The system consists of two main parts:

1. **SDK Telemetry Hooks** - Automatically capture all SDK operations during development
2. **DevTools Viewer** - Web-based interface for visualizing captured data

## Key Capabilities

The viewer offers real-time monitoring including:
- SDK operation tracking as they execute
- Detailed request/response inspection
- Token usage metrics across requests
- Error debugging with full error details
- Side-by-side run comparisons
- Dark/Light theme support

Telemetry hooks automatically capture chat completions, token usage, timing data, errors, function calls, and contextual information like git branch and model details.

## Installation & Setup

Install via npm as a development dependency:
```bash
npm install @openrouter/devtools --save-dev
```

Basic integration requires passing hooks to the OpenRouter client:
```typescript
const sdk = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
  hooks: createOpenRouterDevtools()
});
```

Launch the viewer with:
```bash
openrouter devtools
```

## Configuration

Customizable options include storage location (default: `.devtools/openrouter-generations.json`) and server URL (default: `http://localhost:4983/api/notify`).

## Safety Features

The tool prevents production deployment by throwing errors when `NODE_ENV === 'production'`. Telemetry capture operates asynchronously to avoid blocking SDK operations, and DevTools failures never propagate to SDK calls.
