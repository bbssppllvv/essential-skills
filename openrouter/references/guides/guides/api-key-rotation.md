# API Key Rotation - OpenRouter Documentation

## Overview

OpenRouter's Management API enables programmatic API key rotation without service interruption, supporting security best practices and compliance requirements.

## Why Rotate API Keys?

Regular rotation accomplishes several goals: it "limits the window of exposure if a key is compromised," helps meet compliance standards, enables audit trail tracking, and facilitates access revocation for departing team members or obsolete systems.

## Rotation Strategy

The zero-downtime approach involves three sequential steps: create a new key, migrate applications to use it, then remove the old one. The documentation emphasizes: "Always verify your new key is working in production before deleting the old one."

## Implementation Steps

### Step 1: Create a New Key

You'll need a Management API key from OpenRouter settings. The creation process is available in three formats:

- **TypeScript SDK**: Uses the OpenRouter SDK to create a key with name and spending limit parameters
- **Python**: Makes a POST request to the `/keys/` endpoint with Bearer token authentication
- **TypeScript (fetch)**: Raw fetch-based implementation with identical parameters

All approaches return the new key and its hash for later deletion.

### Step 2: Update Applications

Deploy the new key across your infrastructure through environment variables, secrets managers (AWS Secrets Manager, HashiCorp Vault), or CI/CD pipeline updates. Both keys remain valid during this transition, enabling gradual rollout.

### Step 3: Delete the Old Key

Once migration is complete, remove the old key using the key hash, available in three implementation languages matching the creation step.

## BYOK Advantage

With Bring Your Own Key configuration, "you can rotate your OpenRouter API keys without ever needing to rotate your provider keys." This separation means provider credentials (OpenAI, Anthropic, Google) remain stable while application-facing OpenRouter keys rotate independently through a single management interface.

## Best Practices

- Use descriptive naming with dates or version numbers
- Monitor the Activity page to confirm traffic migration
- Apply spending limits to new keys
- Establish a regular rotation schedule (e.g., quarterly)
- Validate processes in staging environments first
