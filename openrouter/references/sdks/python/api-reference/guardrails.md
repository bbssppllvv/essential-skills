# Guardrails - Python SDK Documentation

## Overview

The Guardrails endpoints enable management of guardrails for the OpenRouter Python SDK. All operations require a provisioning API key for authentication.

**Note:** "The Python SDK and docs are currently in beta. Report issues on GitHub."

## Available Operations

- **list** - List all guardrails
- **create** - Create a new guardrail
- **get** - Retrieve a specific guardrail by ID
- **update** - Modify an existing guardrail
- **delete** - Remove a guardrail
- **list_key_assignments** - View all API key guardrail assignments
- **list_member_assignments** - View all organization member assignments
- **list_guardrail_key_assignments** - View key assignments for a specific guardrail
- **bulk_assign_keys** - Assign multiple API keys to a guardrail
- **list_guardrail_member_assignments** - View member assignments for a guardrail
- **bulk_assign_members** - Assign multiple members to a guardrail
- **bulk_unassign_keys** - Remove multiple API keys from a guardrail
- **bulk_unassign_members** - Remove multiple members from a guardrail

## Key Parameters

**For Creating Guardrails:**
- `name` (required): Guardrail identifier
- `description`: Optional details
- `limit_usd`: Spending cap in USD
- `reset_interval`: Limit reset frequency (daily, weekly, monthly)
- `allowed_providers`: List of provider IDs
- `allowed_models`: Supported model identifiers
- `enforce_zdr`: Zero data retention enforcement flag

**For Assignments:**
- `key_hashes`: Array of API key hashes
- `member_user_ids`: Array of member user identifiers

## Common Error Responses

- 400: Bad Request
- 401: Unauthorized
- 404: Not Found
- 500: Internal Server Error

All operations support optional retry configuration through the `RetryConfig` parameter.
