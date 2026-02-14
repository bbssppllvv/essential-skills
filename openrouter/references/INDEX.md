# OpenRouter Documentation Index

> 178 pages fetched from https://openrouter.ai/docs/ on 2026-02-14

## Getting Started

- [Quickstart](quickstart.md) — Integration guide with cURL, Python, TypeScript, and frameworks
- [Principles](guides/overview/principles.md) — Core design philosophy and approach
- [Models](guides/overview/models.md) — Available models and selection guidance
- [FAQ](faq.md) — Frequently asked questions
- [Enterprise Quickstart](enterprise-quickstart.md) — Enterprise setup and onboarding
- [App Attribution](app-attribution.md) — Branding and attribution guidelines

## Multimodal

- [Overview](guides/overview/multimodal/overview.md) — Multimodal capabilities summary
- [Image Inputs](guides/overview/multimodal/images.md) — Sending images to models
- [Image Generation](guides/overview/multimodal/image-generation.md) — Generating images via API
- [PDF Inputs](guides/overview/multimodal/pdfs.md) — Processing PDF documents
- [Audio Inputs](guides/overview/multimodal/audio.md) — Audio processing
- [Video Inputs](guides/overview/multimodal/videos.md) — Video processing

## Authentication & Security

- [OAuth PKCE](guides/overview/auth/oauth.md) — OAuth 2.0 PKCE flow for user auth
- [API Key Management](guides/overview/auth/management-api-keys.md) — Creating and managing API keys
- [BYOK (Bring Your Own Key)](guides/overview/auth/byok.md) — Using your own provider API keys
- [Report Feedback](guides/overview/report-feedback.md) — Reporting model outputs

## Routing & Model Selection

- [Model Fallbacks](guides/routing/model-fallbacks.md) — Automatic failover between models
- [Provider Routing](guides/routing/provider-selection.md) — Controlling which providers serve requests

### Model Variants

- [Free (:free)](guides/routing/model-variants/free.md) — Zero-cost model access
- [Extended (:extended)](guides/routing/model-variants/extended.md) — Extended context windows
- [Exacto (:exacto)](guides/routing/model-variants/exacto.md) — Exact parameter matching
- [Thinking (:thinking)](guides/routing/model-variants/thinking.md) — Reasoning/chain-of-thought
- [Online (:online)](guides/routing/model-variants/online.md) — Real-time web search
- [Nitro (:nitro)](guides/routing/model-variants/nitro.md) — High-throughput routing

### Routers

- [Auto Router](guides/routing/routers/auto-router.md) — AI-powered model selection (openrouter/auto)
- [Body Builder](guides/routing/routers/body-builder.md) — Natural language to API (openrouter/bodybuilder)
- [Free Models Router](guides/routing/routers/free-models-router.md) — Free inference (openrouter/free)

## Features

- [Presets](guides/features/presets.md) — Reusable LLM configuration profiles
- [Tool & Function Calling](guides/features/tool-calling.md) — Tool use across models
- [Structured Outputs](guides/features/structured-outputs.md) — JSON Schema validated responses
- [Message Transforms](guides/features/message-transforms.md) — Context window compression
- [Zero Completion Insurance](guides/features/zero-completion-insurance.md) — No charge for failed responses
- [Zero Data Retention](guides/features/zdr.md) — Privacy enforcement
- [Guardrails](guides/features/guardrails.md) — Spending and access controls

### Plugins

- [Overview](guides/features/plugins/overview.md) — Available plugins
- [Web Search](guides/features/plugins/web-search.md) — Real-time web search plugin
- [Response Healing](guides/features/plugins/response-healing.md) — Automatic JSON repair

### Broadcast (Observability Integrations)

- [Overview](guides/features/broadcast/overview.md) — Broadcast destinations overview
- [Arize AI](guides/features/broadcast/arize.md)
- [Braintrust](guides/features/broadcast/braintrust.md)
- [ClickHouse](guides/features/broadcast/clickhouse.md)
- [Comet Opik](guides/features/broadcast/opik.md)
- [Datadog](guides/features/broadcast/datadog.md)
- [Grafana Cloud](guides/features/broadcast/grafana.md)
- [Langfuse](guides/features/broadcast/langfuse.md)
- [LangSmith](guides/features/broadcast/langsmith.md)
- [New Relic](guides/features/broadcast/newrelic.md)
- [OpenTelemetry Collector](guides/features/broadcast/otel-collector.md)
- [PostHog](guides/features/broadcast/posthog.md)
- [S3 / S3-Compatible](guides/features/broadcast/s3.md)
- [Sentry](guides/features/broadcast/sentry.md)
- [Snowflake](guides/features/broadcast/snowflake.md)
- [W&B Weave](guides/features/broadcast/weave.md)
- [Webhook](guides/features/broadcast/webhook.md)

## Privacy

- [Data Collection](guides/privacy/data-collection.md) — What data is stored and when
- [Logging](guides/privacy/logging.md) — Provider data retention policies

## Best Practices

- [Latency & Performance](guides/best-practices/latency-and-performance.md) — Optimization tips
- [Prompt Caching](guides/best-practices/prompt-caching.md) — Cross-provider caching
- [Uptime Optimization](guides/best-practices/uptime-optimization.md) — Maximizing availability
- [Reasoning Tokens](guides/best-practices/reasoning-tokens.md) — Configuring thinking tokens

## Guides

- [API Key Rotation](guides/guides/api-key-rotation.md) — Zero-downtime key rotation
- [Crypto API](guides/guides/crypto-api.md) — Cryptocurrency payments via Coinbase
- [MCP Servers](guides/guides/mcp-servers.md) — Model Context Protocol integration
- [Claude Code Integration](guides/guides/claude-code-integration.md) — Using OpenRouter with Claude Code
- [OpenClaw Integration](guides/guides/openclaw-integration.md) — Multi-channel AI agents
- [Provider Integration](guides/guides/for-providers.md) — Provider onboarding
- [Usage Accounting](guides/guides/usage-accounting.md) — Token and cost tracking
- [Activity Export](guides/guides/activity-export.md) — Exporting usage data
- [User Tracking](guides/guides/user-tracking.md) — End-user tracking
- [Distillation](guides/guides/distillation.md) — Model distillation compliance
- [Claude 4.6 Opus Migration](guides/guides/model-migrations/claude-4-6-opus.md) — Migration guide
- [Free Models Router Playground](guides/guides/free-models-router-playground.md)

## Use Cases

- [Organization Management](use-cases/organization-management.md) — Teams, credits, API keys

## Community & Framework Integrations

- [Frameworks Overview](guides/community/frameworks-and-integrations-overview.md) — All integrations
- [Awesome OpenRouter](guides/community/awesome-openrouter.md) — Community projects
- [Effect AI SDK](guides/community/effect-ai-sdk.md)
- [Arize](guides/community/arize.md)
- [LangChain](guides/community/langchain.md) — Python & TypeScript
- [LiveKit](guides/community/livekit.md) — Voice AI agents
- [Langfuse](guides/community/langfuse.md)
- [Mastra](guides/community/mastra.md)
- [OpenAI SDK](guides/community/openai-sdk.md) — Drop-in replacement
- [Anthropic Agent SDK](guides/community/anthropic-agent-sdk.md)
- [PydanticAI](guides/community/pydantic-ai.md)
- [TanStack AI](guides/community/tanstack-ai.md)
- [Vercel AI SDK](guides/community/vercel-ai-sdk.md)
- [Xcode](guides/community/xcode.md) — Apple Intelligence
- [Zapier](guides/community/zapier.md)
- [Infisical](guides/community/infisical.md) — Secrets management

## API Reference

- [Overview](api/reference/overview.md) — Request/response structure, OpenAPI specs
- [Authentication](api/reference/authentication.md) — Bearer token auth
- [Parameters](api/reference/parameters.md) — Sampling and generation params
- [Streaming](api/reference/streaming.md) — SSE streaming implementation
- [Embeddings](api/reference/embeddings.md) — Vector embeddings API
- [Limits](api/reference/limits.md) — Rate limits and quotas
- [Errors & Debugging](api/reference/errors-and-debugging.md) — Error codes and debug mode

### Responses API (Beta)

- [Overview](api/reference/responses/overview.md)
- [Basic Usage](api/reference/responses/basic-usage.md)
- [Reasoning](api/reference/responses/reasoning.md)
- [Tool Calling](api/reference/responses/tool-calling.md)
- [Web Search](api/reference/responses/web-search.md)
- [Error Handling](api/reference/responses/error-handling.md)

## REST API Endpoints

### Chat
- [Send Chat Completion](api/api-reference/chat/send-chat-completion-request.md) — `POST /api/v1/chat/completions`

### Responses
- [Create Response](api/api-reference/responses/create-responses.md) — `POST /api/v1/responses`

### Anthropic Messages
- [Create Message](api/api-reference/anthropic-messages/create-messages.md) — `POST /api/v1/messages`

### OAuth
- [Exchange Auth Code](api/api-reference/o-auth/exchange-auth-code-for-api-key.md) — `POST /api/v1/auth/keys`
- [Create Auth Code](api/api-reference/o-auth/create-auth-keys-code.md) — `POST /api/v1/auth/keys/code`

### Analytics
- [Get User Activity](api/api-reference/analytics/get-user-activity.md) — `GET /api/v1/activity`

### Credits
- [Get Credits](api/api-reference/credits/get-credits.md) — `GET /api/v1/credits`
- [Create Coinbase Charge](api/api-reference/credits/create-coinbase-charge.md) — `POST /api/v1/credits/coinbase`

### Embeddings
- [Create Embeddings](api/api-reference/embeddings/create-embeddings.md) — `POST /api/v1/embeddings`
- [List Embeddings Models](api/api-reference/embeddings/list-embeddings-models.md) — `GET /api/v1/embeddings/models`

### Generations
- [Get Generation](api/api-reference/generations/get-generation.md) — `GET /api/v1/generation`

### Models
- [Get Models Count](api/api-reference/models/list-models-count.md) — `GET /api/v1/models/count`
- [List All Models](api/api-reference/models/get-models.md) — `GET /api/v1/models`
- [List User Models](api/api-reference/models/list-models-user.md) — `GET /api/v1/models/user`

### Endpoints
- [List Model Endpoints](api/api-reference/endpoints/list-endpoints.md) — `GET /api/v1/models/{author}/{slug}/endpoints`
- [Preview ZDR Impact](api/api-reference/endpoints/list-endpoints-zdr.md) — `GET /api/v1/endpoints/zdr`

### Providers
- [List All Providers](api/api-reference/providers/list-providers.md) — `GET /api/v1/providers`

### API Keys
- [List API Keys](api/api-reference/api-keys/list.md) — `GET /api/v1/keys`
- [Create API Key](api/api-reference/api-keys/create-keys.md) — `POST /api/v1/keys`
- [Get API Key](api/api-reference/api-keys/get-key.md) — `GET /api/v1/keys/{hash}`
- [Update API Key](api/api-reference/api-keys/update-keys.md) — `PATCH /api/v1/keys/{hash}`
- [Delete API Key](api/api-reference/api-keys/delete-keys.md) — `DELETE /api/v1/keys/{hash}`
- [Get Current Key](api/api-reference/api-keys/get-current-key.md) — `GET /api/v1/key`

### Guardrails
- [List Guardrails](api/api-reference/guardrails/list-guardrails.md) — `GET /api/v1/guardrails`
- [Create Guardrail](api/api-reference/guardrails/create-guardrail.md) — `POST /api/v1/guardrails`
- [Get Guardrail](api/api-reference/guardrails/get-guardrail.md) — `GET /api/v1/guardrails/{id}`
- [Update Guardrail](api/api-reference/guardrails/update-guardrail.md) — `PATCH /api/v1/guardrails/{id}`
- [Delete Guardrail](api/api-reference/guardrails/delete-guardrail.md) — `DELETE /api/v1/guardrails/{id}`
- [List Key Assignments](api/api-reference/guardrails/list-key-assignments.md) — `GET /api/v1/guardrails/assignments/keys`
- [List Member Assignments](api/api-reference/guardrails/list-member-assignments.md) — `GET /api/v1/guardrails/assignments/members`
- [List Guardrail Key Assignments](api/api-reference/guardrails/list-guardrail-key-assignments.md) — `GET /api/v1/guardrails/{id}/assignments/keys`
- [Bulk Assign Keys](api/api-reference/guardrails/bulk-assign-keys-to-guardrail.md) — `POST /api/v1/guardrails/{id}/assignments/keys`
- [List Guardrail Member Assignments](api/api-reference/guardrails/list-guardrail-member-assignments.md) — `GET /api/v1/guardrails/{id}/assignments/members`
- [Bulk Assign Members](api/api-reference/guardrails/bulk-assign-members-to-guardrail.md) — `POST /api/v1/guardrails/{id}/assignments/members`
- [Bulk Unassign Keys](api/api-reference/guardrails/bulk-unassign-keys-from-guardrail.md) — `POST /api/v1/guardrails/{id}/assignments/keys/remove`
- [Bulk Unassign Members](api/api-reference/guardrails/bulk-unassign-members-from-guardrail.md) — `POST /api/v1/guardrails/{id}/assignments/members/remove`

## TypeScript SDK

- [Overview](sdks/typescript/overview.md) — Installation, setup, type safety
- [Agentic Usage](sdks/agentic-usage.md) — Agent skills for AI coding assistants
- [DevTools](sdks/dev-tools/devtools.md) — Telemetry and visualization

### callModel

- [Overview](sdks/typescript/call-model/overview.md) — callModel function
- [Working with Items](sdks/typescript/call-model/working-with-items.md) — Items-based streaming
- [API Reference](sdks/typescript/call-model/api-reference.md) — Full callModel API
- [Dynamic Parameters](sdks/typescript/call-model/dynamic-parameters.md) — Adaptive params
- [Next Turn Params](sdks/typescript/call-model/next-turn-params.md) — Tool-driven params
- [Stop Conditions](sdks/typescript/call-model/stop-conditions.md) — Execution control
- [Streaming](sdks/typescript/call-model/streaming.md) — Streaming patterns
- [Text Generation](sdks/typescript/call-model/text-generation.md) — Basic text generation
- [Message Formats](sdks/typescript/call-model/message-formats.md) — Format conversion
- [Tools](sdks/typescript/call-model/tools.md) — Tool creation with Zod schemas

### Examples

- [Weather Tool](sdks/typescript/call-model/examples/weather-tool.md) — Complete weather tool
- [Skills Loader](sdks/typescript/call-model/examples/skills-loader.md) — Context injection

### TypeScript API Reference

- [Analytics](sdks/typescript/api-reference/analytics.md)
- [API Keys](sdks/typescript/api-reference/apikeys.md)
- [Chat](sdks/typescript/api-reference/chat.md)
- [Credits](sdks/typescript/api-reference/credits.md)
- [Embeddings](sdks/typescript/api-reference/embeddings.md)
- [Endpoints](sdks/typescript/api-reference/endpoints.md)
- [Generations](sdks/typescript/api-reference/generations.md)
- [Guardrails](sdks/typescript/api-reference/guardrails.md)
- [Models](sdks/typescript/api-reference/models/models.md)
- [OAuth](sdks/typescript/api-reference/oauth.md)
- [Providers](sdks/typescript/api-reference/providers.md)
- [Responses (Beta)](sdks/typescript/api-reference/responses.md)

## Python SDK

- [Overview](sdks/python/overview.md) — Installation, setup, async support

### Python API Reference

- [Analytics](sdks/python/api-reference/analytics.md)
- [API Keys](sdks/python/api-reference/apikeys.md)
- [Chat](sdks/python/api-reference/chat.md)
- [Credits](sdks/python/api-reference/credits.md)
- [Embeddings](sdks/python/api-reference/embeddings.md)
- [Endpoints](sdks/python/api-reference/endpoints.md)
- [Generations](sdks/python/api-reference/generations.md)
- [Guardrails](sdks/python/api-reference/guardrails.md)
- [Models](sdks/python/api-reference/models/models.md)
- [OAuth](sdks/python/api-reference/oauth.md)
- [Providers](sdks/python/api-reference/providers.md)
- [Responses](sdks/python/api-reference/responses.md)
