# Vercel Platform Infrastructure for AI Apps

How Vercel runs your AI code — compute model, function configuration, streaming, caching, pricing, and deployment.

## Table of Contents

1. [Fluid Compute](#fluid-compute)
2. [Function Configuration](#function-configuration)
3. [Streaming on Vercel](#streaming-on-vercel)
4. [@vercel/functions Package](#vercelfunctions-package)
5. [Storage for AI](#storage-for-ai)
6. [Feature Flags for AI Experimentation](#feature-flags-for-ai-experimentation)
7. [Pricing — Active CPU Billing](#pricing--active-cpu-billing)
8. [Edge vs Node.js Runtime](#edge-vs-nodejs-runtime)
9. [Multi-Region Deployment](#multi-region-deployment)
10. [Observability](#observability)
11. [Limits Quick Reference](#limits-quick-reference)

---

## Fluid Compute

Vercel's execution model for Functions — enabled by default on all new projects. A hybrid between serverless and long-running servers.

**How it works:** Multiple requests share a single function instance. While one request waits on I/O (LLM call, DB query), the same instance handles another request's CPU work. This means fewer cold starts, lower latency, and lower cost — especially for AI apps where most time is spent waiting for model responses.

```
Traditional Serverless:
  Request A: [CPU][----LLM wait----][CPU]  → Instance 1
  Request B: [CPU][----LLM wait----][CPU]  → Instance 2 (cold start)

Fluid Compute:
  Request A: [CPU][----LLM wait----][CPU]  → Instance 1
  Request B:      [CPU][--LLM wait--][CPU] → Instance 1 (reused)
```

Enable explicitly if needed:
```json
{ "fluid": true }
```

Key properties:
- **Active CPU billing** — you pay only for CPU execution time, I/O wait is free
- **Bytecode caching** — Node.js 20+ caches compiled bytecode across cold starts (production only)
- **Error isolation** — uncaught exception in one request doesn't crash concurrent requests
- **Scale to zero** — no idle billing

---

## Function Configuration

### maxDuration

| Plan | Default | Max (Fluid) | Max (Legacy) |
|------|---------|-------------|-------------|
| Hobby | 300s | 300s | 60s |
| Pro | 300s | **800s** | 300s |
| Enterprise | 300s | **800s** | 900s |

Set in code:
```typescript
// app/api/chat/route.ts
export const maxDuration = 800; // 13 minutes for long AI streams
```

Or in `vercel.json`:
```json
{
  "functions": {
    "app/api/chat/**/*": { "maxDuration": 800 },
    "app/api/embed/**/*": { "maxDuration": 60 }
  }
}
```

### Memory / CPU Tiers

| Tier | Memory | CPU | Plans |
|------|--------|-----|-------|
| Standard (default) | 2 GB | 1 vCPU | All |
| Performance | 4 GB | 2 vCPUs | Pro, Enterprise |

Configure in Dashboard: Settings > Functions > Advanced Settings > Function CPU.

### Request Body Size Limit: 4.5 MB

Exceeding returns `413 FUNCTION_PAYLOAD_TOO_LARGE`. Workarounds:

1. **Streaming responses** bypass the 4.5 MB response limit entirely
2. **Client uploads** to Vercel Blob (see Storage section) bypass the request limit
3. **Pre-signed URLs** for large media

### Other Limits

- Bundle size: 250 MB (Node.js), 500 MB (Python), 1-4 MB (Edge)
- `/tmp` scratch space: 500 MB (read-only filesystem otherwise)
- File descriptors: 1,024 per instance (shared across concurrent requests)
- Concurrent executions: 30K (Hobby/Pro), 100K+ (Enterprise)
- Burst: 1,000 per 10 seconds per region
- Environment variables: 64 KB total per deployment

---

## Streaming on Vercel

### Node.js Runtime (recommended for AI)

- Governed by `maxDuration` — up to 800s on Pro/Enterprise
- No separate initial response deadline
- Streaming responses bypass the 4.5 MB body limit
- `waitUntil()` works with streaming

### Edge Runtime

- Must begin sending response within **25 seconds**
- Can then stream for up to **300 seconds** total
- If no bytes within 25s → timeout

### AI SDK handles backpressure automatically:

```typescript
export const maxDuration = 800;

export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({ model: openai('gpt-4o'), messages });
  return result.toDataStreamResponse(); // Backpressure handled by SDK
}
```

---

## @vercel/functions Package

```bash
npm i @vercel/functions
```

### waitUntil(promise)

Background work after response is sent — shares function timeout:

```typescript
import { waitUntil } from '@vercel/functions';

export async function POST(req: Request) {
  const response = Response.json({ ok: true });
  waitUntil(logUsage(req)); // Non-blocking background work
  return response;
}
```

### after() — Next.js 15.1+

Same concept, cleaner API for Next.js:

```typescript
import { after } from 'next/server';

export async function POST(req: Request) {
  after(async () => {
    await logToAnalytics(req);
    await sandbox.stop(); // Clean up sandbox after response
  });
  return result.toDataStreamResponse();
}
```

### geolocation(request) / ipAddress(request)

```typescript
import { geolocation, ipAddress } from '@vercel/functions';

const { city, country } = geolocation(request);
const ip = ipAddress(request);
```

### getCache() — Runtime Cache

Regional cache for caching AI responses, search results, etc.:

```typescript
import { getCache } from '@vercel/functions';

const cache = getCache();

// Read
const cached = await cache.get('search:query');

// Write with TTL and tags
await cache.set('search:query', results, {
  ttl: 300,             // 5 minutes
  tags: ['search'],
});

// Invalidate all entries with tag (propagates globally in ~300ms)
await cache.expireTag('search');
```

Limits: 2 MB per item, 128 tags per item, regional (per Vercel region), persistent across deployments.

### attachDatabasePool(pool)

Critical for Fluid Compute — ensures DB connections are released before instance suspends:

```typescript
import { Pool } from 'pg';
import { attachDatabasePool } from '@vercel/functions';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
attachDatabasePool(pool); // Call immediately after creating pool
```

Supports: pg, mysql2, mariadb, mongodb, ioredis, cassandra.

---

## Storage for AI

### Vercel Blob — File Storage

```bash
npm i @vercel/blob
```

Store AI-generated images, audio, PDFs, and handle large user uploads:

```typescript
import { put } from '@vercel/blob';

// Store AI-generated image
const blob = await put('generated/image.png', imageBuffer, {
  access: 'public',
  contentType: 'image/png',
  multipart: true, // For files >100MB
});
// Returns: { url, pathname, contentType, ... }
```

**Client uploads** (bypass 4.5MB body limit):

```typescript
// Client
import { upload } from '@vercel/blob/client';
const blob = await upload(file.name, file, {
  access: 'private',
  handleUploadUrl: '/api/upload',
});

// Server — token handler
import { handleUpload } from '@vercel/blob/client';
export async function POST(request: Request) {
  return Response.json(await handleUpload({
    body: await request.json(),
    request,
    onBeforeGenerateToken: async () => ({
      allowedContentTypes: ['image/jpeg', 'image/png'],
    }),
    onUploadCompleted: async ({ blob }) => {
      await db.update({ fileUrl: blob.url });
    },
  }));
}
```

### Edge Config — Ultra-Low-Latency Reads

```bash
npm i @vercel/edge-config
```

Sub-millisecond reads for runtime config — model selection, prompt switching, IP blocking:

```typescript
import { get } from '@vercel/edge-config';

// In middleware or function — <1ms read, no redeployment needed
const modelConfig = await get('ai_model_config');
// { "default": "gpt-4o", "fallback": "claude-haiku-4-5", "temperature": 0.7 }
```

Writes are via Vercel REST API only (not SDK).

### Upstash Redis (replaces deprecated @vercel/kv)

For rate limiting state, AI response caching, session storage. Install via Vercel Marketplace.

```typescript
import { Redis } from '@upstash/redis';
const redis = Redis.fromEnv();
```

### Vector Databases

Via Vercel Marketplace: Upstash Vector, Neon (pgvector), Pinecone.

---

## Feature Flags for AI Experimentation

```bash
npm i flags @flags-sdk/vercel
```

A/B test AI models, prompts, and features with server-side evaluation:

```typescript
import { flag } from 'flags/next';
import { vercelAdapter } from '@flags-sdk/vercel';

export const aiModel = flag({
  key: 'ai-model',
  adapter: vercelAdapter(),
  options: [
    { value: 'gpt-4o', label: 'GPT-4o' },
    { value: 'claude-sonnet', label: 'Claude Sonnet' },
  ],
  decide: () => 'gpt-4o',
});

// In your AI route
const model = await aiModel();
const response = await generateText({ model });
```

**Prompt experimentation:**
```typescript
export const systemPrompt = flag({
  key: 'system-prompt-variant',
  adapter: vercelAdapter(),
  options: [
    { value: 'concise', label: 'Concise' },
    { value: 'detailed', label: 'Detailed' },
  ],
  decide: () => 'concise',
});
```

Percentage rollouts are configured in the Vercel Dashboard. Supports providers: Vercel (native), LaunchDarkly, Statsig, and custom adapters.

---

## Pricing — Active CPU Billing

With Fluid Compute, you pay for **active CPU time only**. I/O wait (waiting for LLM API, DB queries) is free for CPU billing.

### Example: AI Chat Endpoint

- Total request: 5 seconds
- Active CPU: 200ms (parsing, formatting, processing)
- I/O wait: 4.8s (waiting for LLM response)
- **You pay for 200ms**, not 5s — up to 90% cheaper than wall-clock billing

### Rates (iad1 — US East)

| Metric | Rate |
|--------|------|
| Active CPU | $0.128/CPU-hour |
| Provisioned Memory | $0.0106/GB-hour |
| Invocations | $0.60/million |

Rates vary by region (e.g., São Paulo ~1.7x US East).

### Billing Flow

1. Request arrives → memory billing starts
2. Code executes → CPU billing starts
3. Code calls LLM API → **CPU billing pauses**, memory continues
4. LLM responds → CPU billing resumes
5. Response sent, no in-flight requests → **all billing stops**

---

## Edge vs Node.js Runtime

**Vercel recommends Node.js for AI apps.** Both run on Fluid Compute with Active CPU pricing.

| Feature | Node.js (recommended) | Edge |
|---------|----------------------|------|
| Max duration (Fluid) | 800s (Pro) | 300s streaming |
| Bundle size | 250 MB | 1-4 MB |
| Node.js APIs | Full | Subset |
| File system | /tmp (500 MB) | None |
| Cold starts | Mitigated by bytecode caching | Near-zero |
| DB connections | Full (native drivers) | Limited |
| Default region | iad1 | Closest to user |

Use Edge only for ultra-low latency middleware, feature flags, or request routing.

---

## Multi-Region Deployment

| Plan | Regions |
|------|---------|
| Hobby | 1 |
| Pro | Up to 3 |
| Enterprise | All 20 |

Default: `iad1` (US East). Configure in Dashboard or `vercel.json`:

```json
{
  "regions": ["iad1", "fra1", "hnd1"],
  "functions": {
    "api/eu-data.js": { "regions": ["cdg1"] }
  }
}
```

**Best practice for AI:** Deploy functions near your AI provider's regional endpoint and your database, not near users. CDN handles user proximity.

---

## Observability

### AI Gateway Monitoring

The AI Gateway tab in Vercel Observability shows:
- Requests by model (across 100+ models)
- Time to first token (TTFT)
- Input/output token counts
- Cost per request
- Per-model latency comparison

### Functions Metrics

Invocation count, error rates, CPU throttling, latency breakdown per route.

### Log and Trace Drains

Export to Datadog, Grafana, Splunk, or any OpenTelemetry-compatible platform.

### Notebooks

Save Observability queries as reusable dashboards for ongoing monitoring.

---

## Limits Quick Reference

| Limit | Value |
|-------|-------|
| Request/response body | 4.5 MB (streaming exempt) |
| Max duration (Fluid, Pro) | 800s |
| Memory (Standard) | 2 GB / 1 vCPU |
| Memory (Performance) | 4 GB / 2 vCPUs |
| Concurrent executions | 30K (Hobby/Pro), 100K+ (Ent) |
| /tmp scratch | 500 MB |
| Env vars total | 64 KB |
| Runtime Cache item | 2 MB |
| Edge initial response | 25 seconds |
| Edge max streaming | 300 seconds |
| Bundle (Node.js) | 250 MB |
| Bundle (Edge) | 1-4 MB |
