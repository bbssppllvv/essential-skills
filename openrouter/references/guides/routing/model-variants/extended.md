# Extended Variant Documentation

## Overview

The `:extended` variant feature allows access to model versions featuring expanded context windows, enabling users to work with longer inputs and preserve extended conversation histories.

## Implementation

To utilize this variant, append `:extended` to your model identifier:

```json
{
  "model": "anthropic/claude-sonnet-4.5:extended"
}
```

## Key Benefits

According to the documentation, "Extended variants offer larger context windows than the standard model versions, allowing you to process longer inputs and maintain more conversation history."

This capability proves valuable for applications requiring substantial token capacity or complex multi-turn interactions.
