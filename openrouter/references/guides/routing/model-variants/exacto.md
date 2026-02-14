# Exacto Variant Documentation

## Overview

The Exacto Variant is a routing feature on OpenRouter that directs requests through providers vetted for superior tool-calling performance. As described, it "route[s] requests through OpenRouter-curated providers" with measurably better tool-use success rates.

## Implementation

To use Exacto, append `:exacto` to any supported model slug. The system enforces the curated provider allowlist before standard sorting and fallback logic -- no additional configuration is needed.

### Code Examples

**TypeScript SDK:**
```typescript
const completion = await openRouter.chat.send({
  model: "moonshotai/kimi-k2-0905:exacto",
  messages: [{
    role: "user",
    content: "Draft a concise changelog entry for the Exacto launch.",
  }],
  stream: false,
});
```

**OpenAI SDK (TypeScript):**
```typescript
const completion = await client.chat.completions.create({
  model: "moonshotai/kimi-k2-0905:exacto",
  messages: [{
    role: "user",
    content: "Draft a concise changelog entry for the Exacto launch.",
  }],
});
```

**cURL:**
```shell
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -d '{"model": "moonshotai/kimi-k2-0905:exacto", ...}'
```

## Supported Models

- Kimi K2
- DeepSeek v3.1 Terminus
- GLM 4.6
- GPT-OSS 120B
- Qwen3 Coder

## Provider Selection Criteria

OpenRouter evaluates providers using real traffic data, user preferences, and benchmarks. Selected providers must rank highly on tool-calling accuracy, maintain normal tool-use propensity, and avoid frequent user blacklisting when tools are provided.
