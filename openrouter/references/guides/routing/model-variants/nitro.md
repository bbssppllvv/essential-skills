# Nitro Variant: High-Speed Inference

## Overview

The `:nitro` variant functions as "an alias for sorting providers by throughput." When applied, OpenRouter will route requests to providers capable of delivering the highest token-per-second performance.

## Implementation

To use this feature, append `:nitro` to your model identifier:

```json
{
  "model": "openai/gpt-5.2:nitro"
}
```

## Technical Details

This approach is functionally identical to manually configuring `provider.sort` to `"throughput"` within your API request parameters. For comprehensive information about provider routing and sorting options, consult the Provider Routing documentation.
