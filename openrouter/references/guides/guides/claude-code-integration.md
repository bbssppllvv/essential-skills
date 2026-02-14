# Claude Code Integration with OpenRouter

## Overview

OpenRouter provides a middleware layer between Claude Code and Anthropic's API, delivering reliability enhancements and organizational management capabilities.

## Key Benefits

**Reliability & Failover:** "Anthropic's API occasionally experiences outages or rate limiting. When you route Claude Code through OpenRouter, your requests automatically fail over between multiple Anthropic providers."

**Budget Management:** Teams gain centralized spending controls, credit allocation across developers, and cost overrun prevention.

**Analytics:** The OpenRouter Activity Dashboard tracks usage patterns, real-time costs, and resource consumption by project or team member.

## Installation & Setup

### Installation Options

Users can install Claude Code via:
- Native installer (macOS, Linux, WSL, Windows PowerShell)
- npm package (requires Node.js 18+)

### Configuration

Connect Claude Code to OpenRouter by setting three environment variables:

```bash
export OPENROUTER_API_KEY="<your-openrouter-api-key>"
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"
export ANTHROPIC_API_KEY="" # Must be explicitly empty
```

Alternatively, create `.claude/settings.local.json` in your project root with these settings.

### Verification

Run `/status` within Claude Code to confirm the connection displays the correct OpenRouter base URL.

## Advanced Integrations

**Agent SDK:** The Anthropic Agent SDK uses identical environment variable configuration.

**GitHub Actions:** Pass the OpenRouter API key and set `ANTHROPIC_BASE_URL` in the workflow step.

**Cost Tracking:** Optional statusline scripts display real-time cost metrics during coding sessions.

## Important Notes

- Claude Code with OpenRouter is "only guaranteed to work with the Anthropic first-party provider"
- Previous Anthropic credentials should be cleared via `/logout` before switching
- Source code prompts aren't logged unless explicitly enabled in account settings
