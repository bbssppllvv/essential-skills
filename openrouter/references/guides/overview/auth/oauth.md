# OAuth PKCE: Secure Authentication for OpenRouter

OpenRouter enables seamless user authentication through "Proof Key for Code Exchange (PKCE)," allowing users to connect in a single click.

## Implementation Steps

### Step 1: Direct Users to OpenRouter

Initiate the flow by sending users to OpenRouter's authentication endpoint with these parameters:

- **callback_url**: Your application's return destination
- **code_challenge**: Optional but recommended for enhanced security
- **code_challenge_method**: Either "S256" (SHA-256, recommended) or "plain"

**Example URLs:**
```
https://openrouter.ai/auth?callback_url=<YOUR_SITE_URL>&code_challenge=<CODE_CHALLENGE>&code_challenge_method=S256
```

Users authenticate and authorize your application, then receive redirected back to your site with a `code` parameter.

### Step 2: Exchange Code for API Key

Extract the authorization code from the URL query parameters:

```typescript
const urlParams = new URLSearchParams(window.location.search);
const code = urlParams.get('code');
```

Submit a POST request to `https://openrouter.ai/api/v1/auth/keys`:

```typescript
const response = await fetch('https://openrouter.ai/api/v1/auth/keys', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    code: '<CODE_FROM_QUERY_PARAM>',
    code_verifier: '<CODE_VERIFIER>',
    code_challenge_method: '<METHOD>'
  })
});
const { key } = await response.json();
```

### Step 3: Utilize the API Key

Store the key securely and employ it for subsequent API requests to OpenRouter's chat completions endpoints.

## Code Challenge Generation

For S256 method security, generate a SHA-256 hash of your code verifier:

```typescript
async function createSHA256CodeChallenge(input: string) {
  const encoder = new TextEncoder();
  const data = encoder.encode(input);
  const hash = await crypto.subtle.digest('SHA-256', data);
  return Buffer.from(hash).toString('base64url');
}
```

## Troubleshooting

| Error | Resolution |
|-------|-----------|
| `400 Invalid code_challenge_method` | Ensure matching methods across steps 1-2 |
| `403 Invalid code or code_verifier` | Verify user login status and parameter accuracy |
| `405 Method Not Allowed` | Confirm POST requests via HTTPS |
