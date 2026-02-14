# OAuth PKCE

Connect your users to OpenRouter.

Users can connect to OpenRouter in one click using [Proof Key for Code Exchange (PKCE)](https://oauth.net/2/pkce/).

Here's a step-by-step guide:

## PKCE Guide

### Step 1: Send your user to OpenRouter

To start the PKCE flow, send your user to OpenRouter's `/auth` URL with a `callback_url` parameter pointing back to your site:

**With S256 Code Challenge (Recommended)**

```
https://openrouter.ai/auth?callback_url=<YOUR_SITE_URL>&code_challenge=<CODE_CHALLENGE>&code_challenge_method=S256
```

**With Plain Code Challenge**

```
https://openrouter.ai/auth?callback_url=<YOUR_SITE_URL>&code_challenge=<CODE_CHALLENGE>&code_challenge_method=plain
```

**Without Code Challenge**

```
https://openrouter.ai/auth?callback_url=<YOUR_SITE_URL>
```

The `code_challenge` parameter is optional but recommended.

#### How to Generate a Code Challenge

```typescript
import { Buffer } from 'buffer';

async function createSHA256CodeChallenge(input: string) {
  const encoder = new TextEncoder();
  const data = encoder.encode(input);
  const hash = await crypto.subtle.digest('SHA-256', data);
  return Buffer.from(hash).toString('base64url');
}

const codeVerifier = 'your-random-string';
const generatedCodeChallenge = await createSHA256CodeChallenge(codeVerifier);
```

#### Localhost Apps

If your app is a local-first app or otherwise doesn't have a public URL, it is recommended to test with `http://localhost:3000` as the callback and referrer URLs.

### Step 2: Exchange the code for a user-controlled API key

Extract this code using the browser API:

```typescript
const urlParams = new URLSearchParams(window.location.search);
const code = urlParams.get('code');
```

Then use it to make an API call to `https://openrouter.ai/api/v1/auth/keys` to exchange the code for a user-controlled API key:

```typescript
const response = await fetch('https://openrouter.ai/api/v1/auth/keys', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    code: '<CODE_FROM_QUERY_PARAM>',
    code_verifier: '<CODE_VERIFIER>',
    code_challenge_method: '<CODE_CHALLENGE_METHOD>',
  }),
});

const { key } = await response.json();
```

### Step 3: Use the API key

Store the API key securely within the user's browser or in your own database, and use it to make OpenRouter requests.

**TypeScript SDK**

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: key,
});

const completion = await openRouter.chat.send({
  model: 'openai/gpt-5.2',
  messages: [
    {
      role: 'user',
      content: 'Hello!',
    },
  ],
  stream: false,
});

console.log(completion.choices[0].message);
```

**TypeScript (fetch)**

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${key}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      {
        role: 'user',
        content: 'Hello!',
      },
    ],
  }),
});
```

## Error Codes

| Error | Resolution |
|-------|-----------|
| `400 Invalid code_challenge_method` | Ensure matching challenge method between steps |
| `403 Invalid code or code_verifier` | Verify user login and parameter accuracy |
| `405 Method Not Allowed` | Confirm POST and HTTPS usage |
