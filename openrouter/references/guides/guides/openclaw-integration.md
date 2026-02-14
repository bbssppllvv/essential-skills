# OpenClaw Integration with OpenRouter - Complete Documentation

## Overview

"OpenClaw (formerly Moltbot, formerly Clawdbot) is an open-source AI agent platform that brings conversational AI to multiple messaging channels."

The platform supports Telegram, Discord, Slack, Signal, iMessage, and WhatsApp, allowing deployment of AI agents across multiple protocols simultaneously.

## Setup Methods

### Recommended Approach: Setup Wizard

The simplest configuration method uses the built-in wizard:

```bash
openclaw onboard
```

This guides users through provider selection, API key entry, model selection, and messaging channel configuration.

### Quick CLI Alternative

For experienced users with existing credentials:

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Manual Configuration

### Obtaining API Credentials

Users must navigate to OpenRouter's platform, access the API Keys section, create a new key, and copy the credential (format: `sk-or-...`).

### Configuration File Setup

The API key can be stored in `~/.openclaw/openclaw.json` or as an environment variable:

```json
{
  "env": {
    "OPENROUTER_API_KEY": "sk-or-..."
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/anthropic/claude-sonnet-4.5"
      },
      "models": {
        "openrouter/anthropic/claude-sonnet-4.5": {}
      }
    }
  }
}
```

### Model Selection Examples

**Anthropic Claude:** `openrouter/anthropic/claude-sonnet-4.5`

**Google Gemini:** `openrouter/google/gemini-pro-1.5`

**DeepSeek:** `openrouter/deepseek/deepseek-chat`

**Moonshot Kimi:** `openrouter/moonshotai/kimi-k2.5`

## Model Format and Routing

"OpenClaw uses the format `openrouter/<author>/<slug>` for OpenRouter models."

The `openrouter/openrouter/auto` option provides intelligent routing, directing simple tasks to cost-effective models while reserving capable models for complex reasoning.

## Fallback Configuration

```json
{
  "model": {
    "primary": "openrouter/anthropic/claude-sonnet-4.5",
    "fallbacks": ["openrouter/anthropic/claude-haiku-3.5"]
  }
}
```

This ensures service continuity when primary models become unavailable.

## Secure Credential Management

Auth profiles store API keys in system keychains rather than configuration files:

```json
{
  "auth": {
    "profiles": {
      "openrouter:default": {
        "provider": "openrouter",
        "mode": "api_key"
      }
    }
  }
}
```

Set credentials via CLI:

```bash
openclaw auth set openrouter:default --key "$OPENROUTER_API_KEY"
```

## Monitoring and Troubleshooting

Users can track activity through the OpenRouter Activity Dashboard, viewing requests, costs, and token consumption.

**Common resolution steps:**

- Verify environment variable configuration with `echo $OPENROUTER_API_KEY`
- Validate API key status and account credits at the keys management page
- Confirm model IDs match the `openrouter/<author>/<slug>` format
- Ensure models are registered in configuration

## Advanced Features

### Channel-Specific Models

Different messaging platforms can use different models:

```json
{
  "telegram": {
    "agents": {
      "defaults": {
        "model": {"primary": "openrouter/anthropic/claude-haiku-3.5"}
      }
    }
  },
  "discord": {
    "agents": {
      "defaults": {
        "model": {"primary": "openrouter/anthropic/claude-sonnet-4.5"}
      }
    }
  }
}
```

## Starting the Service

After configuration:

```bash
openclaw gateway run
```
