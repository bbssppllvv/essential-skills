# Frequently Asked Questions

Common questions about OpenRouter

---

## Getting started

**Why should I use OpenRouter?**

OpenRouter provides a unified API to access all the major LLM models on the market with aggregated billing and usage analytics. The platform passes through underlying provider pricing while pooling uptime for better reliability.

**How do I get started with OpenRouter?**

Create an account, add credits on the Credits page, then use either the chat interface or create API keys for the API.

**How do I get support?**

Technical support is available via Discord (#help forum); billing questions go to support@openrouter.ai.

**How do I get billed for my usage on OpenRouter?**

Pricing is displayed per million tokens with different rates for prompt and completion tokens. Costs are deducted from credits based on actual token usage reported by providers.

---

## Pricing and Fees

**What are the fees for using OpenRouter?**

OpenRouter charges a fee when purchasing credits and passes through provider pricing without markup. Crypto payments have a separate fee structure.

**Is there a fee for using my own provider keys (BYOK)?**

Free tier available monthly; subsequent usage charged as a percentage of standard OpenRouter costs, deducted from credits.

---

## Models and Providers

**What LLM models does OpenRouter support?**

Access to a wide variety of LLM models, including frontier models from major AI labs. Complete list available at the [models browser](https://openrouter.ai/models) or via the models API.

**How frequently are new models added?**

Models added as quickly as possible, often through lab partnerships. User requests welcome on Discord.

**What are model variants?**

Static variants (`:free`, `:extended`, `:exacto`, `:thinking`) apply to specific models. Dynamic variants (`:online`, `:nitro`, `:floor`) available for all models and modify routing behavior.

**I am an inference provider, how can I get listed on OpenRouter?**

See requirements at the [Providers page](https://openrouter.ai/docs/guides/guides/for-providers); contact via email.

**What is the expected latency/response time for different models?**

Latency and throughput data shown per model/provider. Use `:nitro` variant to optimize for speed.

**How does model fallback work if a provider is unavailable?**

If a provider returns an error OpenRouter will automatically fall back to the next provider transparently to users.

---

## API Technical Specifications

**What authentication methods are supported?**

Cookie-based (web interface), API keys (Bearer tokens), and Management API keys for programmatic key management.

**How are rate limits calculated?**

Free models limited by purchased credits threshold; free tier users face daily request limits. Documented in [rate limits documentation](https://openrouter.ai/docs/api/reference/limits).

**What API endpoints are available?**

OpenRouter implements the OpenAI API specification for `/completions` and `/chat/completions` endpoints plus additional endpoints like `/api/v1/models`.

**What are the supported formats?**

The API supports text, images, and PDFs as URLs or base64 encoded data.

**How does streaming work?**

Streaming uses server-sent events (SSE) for real-time token delivery via the `stream: true` parameter.

**What SDK support is available?**

OpenRouter is a drop-in replacement for OpenAI compatible with standard SDKs and frameworks.

---

## Privacy and Data Logging

**What data is logged during API use?**

We log basic request metadata (timestamps, model used, token counts). Prompt and completion are not logged by default. Opt-in logging available for 1% discount.

**What data is logged during Chatroom use?**

Same privacy as API. All conversations in the chatroom are stored locally on your device and don't sync across devices.

**What third-party sharing occurs?**

OpenRouter proxies requests to providers while working to prevent logging/training use. Non-logging providers automatically routed unless training toggle enabled.

---

## Credit and Billing Systems

**What purchase options exist?**

Credit system in US dollars with manual top-up or auto-replenishment options.

**Do credits expire?**

Per our terms, we reserve the right to expire unused credits after one year of purchase.

**My credits haven't showed up in my account**

Allow up to one hour for Stripe processing; contact support@openrouter.ai if unresolved or for crypto payments.

**What's the refund policy?**

Refunds for unused Credits may be requested within twenty-four (24) hours from the time the transaction was processed. Unused credits beyond 24 hours become non-refundable; platform fees non-refundable; crypto never refundable.

**How to monitor credit usage?**

The Activity page allows users to view their historic usage and filter by model, provider, and API key. Credits API available for live balance data.

**What free tier options exist?**

New users receive small free allowance. Free models available with low rate limits and automatic router option.

**How do volume discounts work?**

OpenRouter does not currently offer volume discounts but exceptional cases considered via email.

**What payment methods are accepted?**

We accept all major credit cards, AliPay and cryptocurrency payments in USDC with PayPal integration in development.

**How does OpenRouter make money?**

We charge a small fee when purchasing credits while never marking up provider pricing.

---

## Account Management

**How can I delete my account?**

Go to Settings > Manage Account > Security tab to delete. Unused credits cannot be reclaimed.

**How does team access work?**

Information available in [organization management documentation](https://openrouter.ai/docs/use-cases/organization-management).

**What analytics are available?**

Our activity dashboard provides real-time usage metrics with custom reports available upon request.

**How can I contact support?**

Account/billing: support@openrouter.ai; technical: Discord community.

**How can I file a bug report or change request for OpenRouter?**

Post in Discord.
