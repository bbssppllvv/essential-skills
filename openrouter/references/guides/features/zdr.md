# Zero Data Retention Documentation

## Overview

"Zero Data Retention (ZDR) means that a provider will not store your data for any period of time." OpenRouter enables users to restrict their API calls to only ZDR-compliant endpoints through account settings.

## Key Distinctions

OpenRouter differentiates between two privacy concerns:

1. **Data Retention** -- whether providers store user data
2. **Training** -- whether providers use data to train their models

The platform notes that some endpoints retain data without training on it (for abuse detection or legal compliance), while ZDR endpoints do neither.

## Policy Management

OpenRouter maintains endpoint-specific policies rather than relying solely on provider defaults. The documentation emphasizes that "a provider's general policy may differ from the specific policy for a given endpoint." When policies cannot be verified, the system takes a conservative approach by assuming both retention and training occur.

## Per-Request Enforcement

Developers can enforce ZDR on individual API calls using the `zdr` parameter:

```json
{
  "model": "gpt-4",
  "messages": [...],
  "provider": {
    "zdr": true
  }
}
```

This parameter operates on an "OR" basis with account-wide settings--enabling ZDR at either level activates protection.

## Caching Exception

"In-memory caching of prompts is *not* considered 'retaining' data," allowing cached endpoints to function under ZDR policies.
