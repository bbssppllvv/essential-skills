---
title: Infisical
description: OpenRouter API key management and automatic rotation with Infisical
---

# Infisical Integration

Infisical is a secrets management platform that helps teams securely store, sync, and rotate secrets across their infrastructure. With Infisical's OpenRouter integration, you can automatically rotate your API keys on a schedule, ensuring your credentials stay secure with zero-downtime rotation.

## Prerequisites

You need an OpenRouter **Management API key**. These are special keys used only for key management operations (create, list, delete keys) and cannot be used for model completion requests.

### Creating a Management API Key

1. Navigate to [OpenRouter Settings](https://openrouter.ai/settings)
2. Go to the **Management API Keys** section
3. Click **Create New Key**
4. Complete the key creation flow
5. Copy the generated Management API key and store it securely -- you will need it when creating the Infisical connection

## Setup

### Step 1: Create the OpenRouter Connection in Infisical

1. In the Infisical dashboard, navigate to **Organization Settings > App Connections**
2. Click **Add Connection** and select **OpenRouter**
3. Enter your Management API Key, an optional description, and a connection name
4. Click **Create** -- Infisical will validate the key against OpenRouter's API

### Step 2: Configure Automatic Key Rotation

1. Navigate to your **Secret Manager Project's Dashboard** in Infisical
2. Select **Add Secret Rotation** from the actions dropdown
3. Choose the **OpenRouter API Key** option

## Configuration Parameters

### Scheduling

| Parameter | Description |
|---|---|
| **Auto-Rotation Enabled** | Toggle automatic scheduled rotation on or off |
| **Rotate At** | Local time of day when rotation executes |
| **Rotation Interval** | Number of days between rotations |

### Connection & Key Properties

| Parameter | Description | Required |
|---|---|---|
| **OpenRouter Connection** | Selects which Management API key connection to use | Yes |
| **Key Name** | Display name for the key in OpenRouter | Yes |
| **Limit** | Optional USD spending cap for the key | No |
| **Limit Reset Frequency** | How often the spending limit resets: daily, weekly, or monthly | No |
| **Include BYOK in Limit** | Whether usage from your own provider keys counts toward the spending limit | No |

### Secret Storage

| Parameter | Description |
|---|---|
| **Map to Secret Name** | The Infisical secret path where the rotated key will be stored |

## BYOK Limit Behavior

The **Include BYOK in limit** option controls whether Bring Your Own Key (BYOK) usage counts toward your key's spending limit:

- **When disabled**: Only OpenRouter credit usage counts toward the limit and BYOK usage is tracked separately.
- **When enabled**: Usage from your own provider keys is included in the limit.

## Additional Resources

- [Infisical OpenRouter Connection Guide](https://infisical.com/docs/integrations/app-connections/openrouter)
- [Infisical API Key Rotation Documentation](https://infisical.com/docs)
- [OpenRouter Management Keys](https://openrouter.ai/settings)
- [OpenRouter Quick Start](https://openrouter.ai/docs/quickstart)
