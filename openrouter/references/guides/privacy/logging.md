# Logging | Provider Data Retention Policies

## Overview

OpenRouter's logging and data retention practices vary by provider. Users can control which providers handle their data through account settings.

## Key Points

**Training Policies**
Each provider maintains distinct data handling policies. OpenRouter allows users to "opt out of training" in account settings to restrict routing to providers that train on user data. This setting applies separately to paid and free models.

Users can also filter individual requests to use only providers meeting specific data policies through the privacy settings dashboard.

**Data Retention and Logging**
Providers implement their own retention policies, often for compliance. OpenRouter displays these policies but doesn't use them as routing criteria. Users can independently choose to avoid providers not meeting their data retention requirements.

Provider terms of service are available on individual provider pages and in the documentation.

## Enterprise EU Option

Enterprise customers can enable EU in-region routing. This keeps prompts and completions within the European Union by using the base URL `https://eu.openrouter.ai`.

To check available models for EU routing, call `/api/v1/models/user` through the EU domain. Interested parties should contact the enterprise team at their inquiry form.
