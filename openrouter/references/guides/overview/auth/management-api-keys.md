# Management API Keys Documentation

## Overview

OpenRouter enables programmatic API key management through dedicated endpoints, allowing developers to create, read, update, and delete keys automatically. As stated in the documentation, this feature serves to "manage your API keys, enabling key creation and management for applications that need to distribute or rotate keys automatically."

## Key Setup

To begin using key management capabilities:

1. Navigate to the Management API Keys settings page
2. Generate a new Management API key
3. Use this key exclusively for administrative operations (not for completion endpoint calls)

## Primary Use Cases

The documentation identifies three main scenarios:

- **SaaS Platforms**: Automatically generate unique keys for each customer
- **Security Rotation**: Periodically refresh keys to maintain compliance standards
- **Usage Control**: Monitor consumption and disable keys exceeding thresholds with optional daily/weekly/monthly reset periods

## API Endpoints

All management operations use the `/api/v1/keys` endpoint requiring a Management API key in the Authorization header.

## Implementation Examples

The documentation provides code samples in TypeScript (SDK and fetch), and Python demonstrating:

- Listing keys with pagination (`offset` parameter)
- Creating new keys with optional credit limits
- Retrieving specific key details by hash
- Updating key properties (name, disabled status, limit reset frequency)
- Deleting keys

## Response Structure

Responses contain JSON objects with key metadata including creation/update timestamps, usage statistics (general and BYOK-specific), limits, and reset configurations.
