# Infisical Integration for OpenRouter API Key Management

## Overview

Infisical serves as a secrets management platform enabling teams to securely store, synchronize, and rotate secrets across infrastructure. The platform's OpenRouter integration facilitates "automatic rotation of API keys on a schedule" while maintaining zero-downtime operations.

## Prerequisites

Users must obtain an OpenRouter Management API key before proceeding. These specialized keys are "used only for key management operations (create, list, delete keys) and cannot be used for model completion requests."

### Obtaining a Management API Key

Access the OpenRouter Settings Management API Keys section, create a new key, and store the generated credential securely for use in Infisical configuration.

## Connection Setup

Navigate to Infisical's Organization Settings and App Connections section. Select Add Connection, choose OpenRouter, and input your Management API Key along with a connection name. Infisical validates the credentials against OpenRouter's API.

## Rotation Configuration

### Key Rotation Parameters

The configuration includes several essential settings:

- **Auto-Rotation**: Controls whether keys rotate automatically according to the specified interval
- **Rotation Schedule**: Sets the local time and interval (in days) for rotation execution
- **Connection Selection**: Designates which OpenRouter connection manages key operations
- **Key Name**: Display name for the key in OpenRouter
- **Spending Limits**: Optional USD limit with daily, weekly, or monthly reset cycles
- **BYOK Integration**: Determines whether Bring Your Own Key usage counts toward spending limits

### Secret Storage

Specify the Infisical secret name where rotated API keys will be stored and accessed.

## BYOK and Spending Limits

BYOK allows using personal provider API keys (OpenAI, Anthropic, etc.) while paying providers directly. The "Include BYOK in limit" setting controls whether such usage counts toward the key's spending limit. When disabled, only OpenRouter credits are tracked; when enabled, both usage types are combined.

## Additional Resources

- Infisical OpenRouter Connection documentation
- Infisical OpenRouter API Key Rotation documentation
- OpenRouter Management Keys guide
- OpenRouter Quick Start Guide
