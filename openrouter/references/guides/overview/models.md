# OpenRouter Models Documentation

## Overview

OpenRouter provides unified access to over 400 AI models through a single API endpoint. Users can explore models via the website, API, or RSS feed for updates on new additions.

## Models API Standard

The Models API offers standardized JSON responses with comprehensive metadata for all available language models, cached at the edge for production reliability.

### API Response Schema

**Root Structure:**
```json
{
  "data": [
    /* Array of Model objects */
  ]
}
```

### Model Object Fields

| Field | Type | Purpose |
|-------|------|---------|
| `id` | string | Unique identifier for API requests |
| `canonical_slug` | string | Permanent, unchanging model slug |
| `name` | string | Display name |
| `created` | number | Unix timestamp of addition date |
| `description` | string | Capability overview |
| `context_length` | number | Token limit |
| `architecture` | object | Technical specifications |
| `pricing` | object | Cost structure (USD) |
| `top_provider` | object | Primary provider details |
| `per_request_limits` | data | Rate limiting info |
| `supported_parameters` | array | Compatible API parameters |

### Architecture Details

The architecture object specifies input/output modalities, tokenization method, and instruction format.

### Pricing Structure

Costs are denominated in USD per token or request. The pricing object includes fields for prompts, completions, requests, images, web search, reasoning tokens, and caching operations.

### Supported Parameters

Models support OpenAI-compatible parameters including function calling, temperature control, sampling methods, reasoning modes, structured outputs, and penalties for frequency and presence.

**Important Note:** "Token counts (and therefore costs) will vary between models, even when inputs and outputs are the same" due to different tokenization approaches.
