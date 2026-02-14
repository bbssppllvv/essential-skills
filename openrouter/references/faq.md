# OpenRouter FAQ - Complete Documentation

## Getting Started

**Why use OpenRouter?**
OpenRouter provides unified API access to major LLM models with aggregated billing, analytics, and provider pass-through pricing without markup. It offers improved uptime through provider fallbacks while maintaining direct provider rates.

**Getting Started Steps**
Create an account, add credits on the Credits page, then access the chat interface or create API keys for programmatic use. Pricing varies by model and provider per million tokens.

**Support Channels**
Technical support is available through the Discord community #help forum. Billing and account questions should be directed to support@openrouter.ai.

**Billing Overview**
"For each model we have the pricing displayed per million tokens. There is usually a different price for prompt and completion tokens." Costs are calculated and deducted from credits based on actual token usage, viewable in the Activity tab.

---

## Pricing and Fees

**Credit Purchase Fees**
OpenRouter charges fees when purchasing credits (specific rates apply for Stripe and cryptocurrency payments). No markup exists on inference pricing—users pay provider rates directly.

**Bring Your Own Key (BYOK)**
The first monthly BYOK requests are free; subsequent usage incurs a percentage-based fee deducted from OpenRouter credits, enabling direct provider cost management.

---

## Models and Providers

**Available Models**
"OpenRouter provides access to a wide variety of LLM models, including frontier models from major AI labs." Browse the models page or use the models API for complete listings.

**Model Variants**
Static variants (`:free`, `:extended`, `:exacto`, `:thinking`) apply to specific models. Dynamic variants (`:online`, `:nitro`, `:floor`) work across all models, adjusting routing behavior and optimization focus.

**Provider Integration**
Inference providers can apply for listing by reviewing requirements on the Providers page or contacting OpenRouter via email.

**Latency and Fallbacks**
Each model displays latency and throughput metrics across providers. If a provider fails, "OpenRouter will automatically fall back to the next provider" transparently, enhancing production reliability.

---

## API Technical Specifications

**Authentication Methods**
Three methods are supported: cookie-based (web interface), API keys (Bearer tokens), and Management API keys for programmatic key administration.

**Rate Limits**
Free model limits depend on purchased credits. With sufficient credit purchases, free models allow higher daily request limits; otherwise, standard free tier restrictions apply.

**API Endpoints**
OpenRouter implements OpenAI API specifications for `/completions` and `/chat/completions` endpoints, plus additional endpoints like `/api/v1/models`.

**Supported Formats**
"The API supports text, images, and PDFs." Images can be URLs or base64-encoded; PDFs work as URLs or base64 data across all models.

**Streaming**
Streaming uses server-sent events (SSE) for real-time token delivery via the `stream: true` request parameter.

**SDK Support**
OpenRouter functions as a drop-in OpenAI replacement, supporting any OpenAI-compatible SDKs and multiple frameworks/integrations.

---

## Privacy and Data Logging

**Data Collection**
"We log basic request metadata (timestamps, model used, token counts). Prompt and completion are not logged by default." An opt-in setting provides a 1% usage discount in exchange for logging prompts and completions.

**Chatroom Privacy**
Chatroom conversations are stored locally on devices without cross-device synchronization. Export/import functionality is available through settings.

**Third-Party Sharing**
OpenRouter proxies requests to providers and negotiates no-logging agreements when possible. Providers with logging policies require explicit opt-in via privacy settings to route requests.

---

## Credit and Billing Systems

**Purchase Options**
OpenRouter uses a dollar-denominated credit system with manual top-up or automatic replenishment options.

**Credit Expiration**
"Per our terms, we reserve the right to expire unused credits after one year of purchase."

**Missing Credits**
Stripe delays may require up to one hour; check for receipt emails and successful charges. Reach out to support@openrouter.ai for unresolved issues. Cryptocurrency payments require email support.

**Refund Policy**
"Refunds for unused Credits may be requested within twenty-four (24) hours from the time the transaction was processed." Platform fees are non-refundable; cryptocurrency refunds are unavailable.

**Usage Monitoring**
The Activity page filters usage by model, provider, and API key. The credits API provides live balance information.

**Free Tier**
New users receive limited free access. Free models have low rate limits and aren't suitable for production. Purchasing credits increases free model limits significantly.

**Volume Discounts**
Currently unavailable, though exceptional use cases may qualify—contact OpenRouter via email.

**Payment Methods**
"We accept all major credit cards, AliPay and cryptocurrency payments in USDC." PayPal integration is planned.

**Revenue Model**
OpenRouter charges small credit purchase fees while maintaining provider pricing without markup.

---

## Account Management

**Account Deletion**
Navigate to Settings > Manage Account > Security tab to delete accounts. Unused credits cannot be recovered post-deletion.

**Team Access**
"Organization management information can be found in our organization management documentation."

**Analytics**
The Activity dashboard provides real-time usage metrics; additional reports available upon request.

**Support Contact**
Email support@openrouter.ai for account and billing inquiries.

**Bug Reports**
Post bug reports and change requests in the Discord community.
