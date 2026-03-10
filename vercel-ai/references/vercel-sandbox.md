# Vercel Sandbox — Code Execution for AI Agents

Ephemeral Firecracker microVMs for running untrusted or AI-generated code in isolation. Each sandbox gets its own kernel, filesystem, and network namespace.

## Table of Contents

1. [Quick Start](#quick-start)
2. [SDK API Reference](#sdk-api-reference)
3. [AI Agent Integration Patterns](#ai-agent-integration-patterns)
4. [Snapshots](#snapshots)
5. [Network Policies & Credential Brokering](#network-policies--credential-brokering)
6. [Authentication](#authentication)
7. [System Specs & Pricing](#system-specs--pricing)

---

## Quick Start

```bash
npm i @vercel/sandbox
```

```typescript
import { Sandbox } from '@vercel/sandbox';

const sandbox = await Sandbox.create({ runtime: 'node24', timeout: 30_000 });

await sandbox.writeFiles([
  { path: 'index.js', content: Buffer.from('console.log("Hello from sandbox")') },
]);

const result = await sandbox.runCommand('node', ['index.js']);
console.log(await result.stdout()); // "Hello from sandbox"

await sandbox.stop();
```

Authentication: on Vercel → automatic OIDC. Local → `vercel link && vercel env pull` (token expires after 12 hours).

---

## SDK API Reference

### Sandbox.create(params?)

```typescript
const sandbox = await Sandbox.create({
  runtime: 'node24',           // 'node24' (default), 'node22', 'python3.13'
  resources: { vcpus: 4 },     // 1, 2 (default), 4, or 8
  ports: [3000],               // Up to 4 exposed ports
  timeout: 300_000,            // ms, default 5 min
  networkPolicy: 'allow-all',  // 'allow-all', 'deny-all', or object
  env: { NODE_ENV: 'production' },
  source: { type: 'snapshot', snapshotId: 'snap_abc' }, // Optional
});
```

Implements `AsyncDisposable`:
```typescript
await using sandbox = await Sandbox.create(); // Auto-stop at scope exit
```

### sandbox.runCommand(cmd, args?, opts?)

```typescript
const result = await sandbox.runCommand('node', ['-e', 'console.log(42)']);
console.log(result.exitCode); // 0
console.log(await result.stdout()); // "42"

// Detached (non-blocking)
const cmd = await sandbox.runCommand({
  cmd: 'npm', args: ['run', 'dev'],
  detached: true,
  stdout: process.stdout,
});
// Later: await cmd.wait();
```

Options: `cwd`, `env`, `sudo`, `detached`, `stdout`/`stderr` (Writable streams), `signal`.

### File Operations

```typescript
// Write files
await sandbox.writeFiles([
  { path: 'app.js', content: Buffer.from('...') },
  { path: 'data/config.json', content: Buffer.from(JSON.stringify(config)) },
]);

// Create directory
await sandbox.mkDir('assets');

// Read file as Buffer
const buffer = await sandbox.readFileToBuffer({ path: 'output.json' });

// Read file as stream
const stream = await sandbox.readFile({ path: 'large-file.csv' });

// Download to local filesystem
await sandbox.downloadFile(
  { path: 'report.pdf' },
  { path: '/tmp/report.pdf' },
  { mkdirRecursive: true },
);
```

### Live Preview

```typescript
const sandbox = await Sandbox.create({ ports: [3000] });
await sandbox.runCommand({ cmd: 'npx', args: ['next', 'dev'], detached: true });
const url = sandbox.domain(3000); // "https://<subdomain>.vercel.run"
```

### Other Methods

| Method | Description |
|--------|-------------|
| `sandbox.stop({ blocking? })` | Terminate the microVM |
| `sandbox.snapshot({ expiration? })` | Capture state (stops sandbox) |
| `sandbox.extendTimeout(ms)` | Extend sandbox lifetime |
| `sandbox.updateNetworkPolicy(policy)` | Change firewall rules at runtime |
| `Sandbox.get({ sandboxId })` | Rehydrate active sandbox by ID |
| `Sandbox.list({ projectId? })` | List sandboxes for a project |

### Command Methods

| Method | Description |
|--------|-------------|
| `command.logs()` | AsyncGenerator of `{ stream, data }` log entries |
| `command.wait()` | Block until detached command finishes |
| `command.output('both')` | Get stdout+stderr as string |
| `command.kill(signal?)` | Terminate the process |

---

## AI Agent Integration Patterns

### Pattern A: Sandbox as an AI SDK Tool

```typescript
import { Sandbox } from '@vercel/sandbox';
import { tool } from 'ai';
import { z } from 'zod';

const sandbox = await Sandbox.create({ runtime: 'node22', timeout: 60_000 });

const runCode = tool({
  description: 'Run JavaScript code in an isolated sandbox',
  parameters: z.object({
    code: z.string(),
    packages: z.array(z.string()).optional(),
  }),
  execute: async ({ code, packages }) => {
    if (packages?.length) {
      await sandbox.runCommand('npm', ['install', ...packages]);
    }
    const result = await sandbox.runCommand('node', ['-e', code]);
    return {
      output: await result.stdout(),
      errors: await result.stderr(),
      exitCode: result.exitCode,
    };
  },
});
```

### Pattern B: Pre-built Code Execution Tool

```bash
npm i ai-sdk-tool-code-execution
```

```typescript
import { executeCode } from 'ai-sdk-tool-code-execution';
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

const result = await generateText({
  model: openai('gpt-4o'),
  prompt: 'Calculate the first 20 Fibonacci numbers',
  tools: { executeCode: executeCode({ debug: true }) },
});
```

Uses `python3.13` runtime. Requires explicit `print()` for output.

### Cleanup with after()

```typescript
import { after } from 'next/server';

export async function POST(req: Request) {
  const sandbox = await Sandbox.create();
  // ... use sandbox ...
  after(async () => { await sandbox.stop(); });
  return result.toDataStreamResponse();
}
```

Or with `try/finally`, or `await using` (AsyncDisposable).

---

## Snapshots

Capture full filesystem state — installed packages, configs, generated files. The sandbox stops automatically after snapshot.

```typescript
// Create snapshot (sandbox stops after this)
const snapshot = await sandbox.snapshot({
  expiration: 14 * 24 * 60 * 60 * 1000, // 14 days (default: 30 days, 0 = never)
});

// Restore from snapshot — sub-second startup, deps already installed
const newSandbox = await Sandbox.create({
  source: { type: 'snapshot', snapshotId: snapshot.snapshotId },
});
```

Use cases: skip `npm install` on every run, checkpoint long tasks, share environments.

```typescript
// Manage snapshots
const { json: { snapshots } } = await Snapshot.list();
const snap = await Snapshot.get({ snapshotId: 'snap_abc' });
await snap.delete();
```

---

## Network Policies & Credential Brokering

### Modes

**`'allow-all'`** (default) — full internet access.

**`'deny-all'`** — blocks all egress including DNS.

**Custom rules:**
```typescript
await sandbox.updateNetworkPolicy({
  allow: ['registry.npmjs.org', 'api.openai.com', '*.github.com'],
  subnets: { deny: ['10.0.0.0/8'] }, // Block private network
});
```

### Credential Brokering (Pro/Enterprise)

Inject HTTP headers into outbound requests without exposing secrets to sandbox code. Credentials exist only in the firewall layer outside the VM — even a compromised agent can't exfiltrate them.

```typescript
await sandbox.updateNetworkPolicy({
  allow: {
    'ai-gateway.vercel.sh': [{
      transform: [{ headers: { 'x-api-key': 'sk-secret-key' } }],
    }],
    '*.github.com': [{
      transform: [{ headers: { 'Authorization': 'Bearer ghp_xxxxx' } }],
    }],
  },
});
```

### Multi-Phase Workflow

```typescript
// Phase 1: Install deps with internet
const sandbox = await Sandbox.create({ networkPolicy: 'allow-all' });
await sandbox.runCommand('npm', ['install']);

// Phase 2: Lock down before running untrusted code
await sandbox.updateNetworkPolicy('deny-all');
await sandbox.runCommand('node', ['-e', untrustedCode]);
```

---

## Authentication

| Scenario | Method |
|----------|--------|
| Local development | OIDC: `vercel link && vercel env pull` (12h token) |
| Deployed on Vercel | OIDC automatic — zero config |
| External CI/CD | Access token: `VERCEL_TOKEN` + `VERCEL_TEAM_ID` + `VERCEL_PROJECT_ID` |

---

## System Specs & Pricing

### Specs

| Resource | Value |
|----------|-------|
| OS | Amazon Linux 2023 |
| Runtimes | `node24` (default), `node22`, `python3.13` |
| vCPUs | 1, 2 (default), 4, or 8 |
| Memory per vCPU | 2 GB (max 16 GB at 8 vCPUs) |
| Max ports | 4 |
| Working directory | `/vercel/sandbox` |
| User | `vercel-sandbox` with `sudo` |

### Timeouts & Concurrency

| Plan | Max Duration | Concurrent Sandboxes |
|------|-------------|---------------------|
| Hobby | 45 min | 10 |
| Pro | 5 hours | 2,000 |
| Enterprise | 5 hours | 2,000 |

### Pricing (Pro/Enterprise, iad1)

| Metric | Rate |
|--------|------|
| Active CPU | $0.128/hour |
| Memory | $0.0106/GB-hour |
| Creations | $0.60/million |
| Snapshot storage | $0.08/GB-month |

Active CPU billing — I/O wait is free. Hobby: free within limits.
