# Distillation: Compliance with Provider and Model Creator Policies

## Overview

Model distillation involves training smaller, efficient models using outputs from larger models. OpenRouter helps ensure compliance with provider policies regarding distillation by tracking which models permit their outputs for training purposes.

## Key Concept

As stated in the documentation, "some model providers and creators explicitly prohibit using their model outputs to train other models, while others allow it." OpenRouter addresses this through the `is_trainable_text` property, which identifies models whose authors permit text distillation.

## Finding Distillable Models

Users can locate compliant models in two ways:

**1. Web Interface**
- Navigate to the Models page with the distillable filter enabled
- Filter for "Yes" to display only models permitting distillation

**2. Programmatic Approach**
Use the `enforce_distillable_text` parameter set to `true` in API requests to restrict routing to compliant models only.

## API Implementation Examples

The documentation provides code samples in TypeScript (SDK and fetch), Python, and cURL. Each example demonstrates setting the provider parameter:

```json
{
  "enforce_distillable_text": true
}
```

## Important Consideration

OpenRouter emphasizes that "distillation information [is provided] on a best-effort basis" and recommends verifying specific license terms for your particular use case, as requirements vary by application.

## Practical Applications

- Creating training datasets from model outputs
- Building specialized smaller models from larger teachers
- Maintaining programmatic compliance across API requests
