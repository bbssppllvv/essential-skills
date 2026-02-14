# API Key Rotation

Secure Key Management for OpenRouter

---

Regular API key rotation is a security best practice that limits the impact of compromised credentials. OpenRouter's Management API makes it easy to rotate keys programmatically without service interruption.

## Why Rotate API Keys?

Rotating API keys regularly helps protect your applications by:

- **Limiting exposure**: Reducing the window of vulnerability if a key is compromised
- **Meeting compliance requirements**: Satisfying credential management policies
- **Enabling clean audit trails**: Tracking key usage over defined periods
- **Revoking access**: Allowing you to revoke access for former team members or deprecated systems

## Zero-Downtime Rotation Process

A zero-downtime key rotation follows three steps:

1. **Create a new key**
2. **Update your applications** to use the new key
3. **Delete the old key** once all systems have migrated

> **Important**: Always verify your new key is working in production before deleting the old one.

### Step 1: Create a New Key

#### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: 'your-management-key',
});

const newKey = await openRouter.apiKeys.create({
  name: 'Production Key - Rotated 2025-01',
  limit: 1000,
});

console.log('New key created:', newKey.data.key);
console.log('Key hash:', newKey.data.hash);
```

#### Python

```python
import requests

MANAGEMENT_API_KEY = "your-management-key"
BASE_URL = "https://openrouter.ai/api/v1/keys"

response = requests.post(
    f"{BASE_URL}/",
    headers={
        "Authorization": f"Bearer {MANAGEMENT_API_KEY}",
        "Content-Type": "application/json"
    },
    json={
        "name": "Production Key - Rotated 2025-01",
        "limit": 1000
    }
)

data = response.json()
print(f"New key created: {data['data']['key']}")
print(f"Key hash: {data['data']['hash']}")
```

#### TypeScript (fetch)

```typescript
const MANAGEMENT_API_KEY = 'your-management-key';
const BASE_URL = 'https://openrouter.ai/api/v1/keys';

const response = await fetch(BASE_URL, {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${MANAGEMENT_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    name: 'Production Key - Rotated 2025-01',
    limit: 1000,
  }),
});

const { data } = await response.json();
console.log('New key created:', data.key);
console.log('Key hash:', data.hash);
```

### Step 2: Deploy Updated Credentials

Both old and new keys remain valid during the transition period. Update your credentials across:

- Environment variables
- Secrets managers (AWS Secrets Manager, HashiCorp Vault)
- CI/CD systems
- Application configuration

### Step 3: Delete the Old Key

Once all systems have migrated to the new key, delete the old one using the key hash returned from the creation response.

#### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: 'your-management-key',
});

const oldKeyHash = 'hash-of-old-key';
await openRouter.apiKeys.delete(oldKeyHash);

console.log('Old key deleted successfully');
```

#### Python

```python
import requests

MANAGEMENT_API_KEY = "your-management-key"
BASE_URL = "https://openrouter.ai/api/v1/keys"

old_key_hash = "hash-of-old-key"
response = requests.delete(
    f"{BASE_URL}/{old_key_hash}",
    headers={
        "Authorization": f"Bearer {MANAGEMENT_API_KEY}",
        "Content-Type": "application/json"
    }
)

print("Old key deleted successfully")
```

#### TypeScript (fetch)

```typescript
const MANAGEMENT_API_KEY = 'your-management-key';
const BASE_URL = 'https://openrouter.ai/api/v1/keys';

const oldKeyHash = 'hash-of-old-key';
await fetch(`${BASE_URL}/${oldKeyHash}`, {
  method: 'DELETE',
  headers: {
    Authorization: `Bearer ${MANAGEMENT_API_KEY}`,
    'Content-Type': 'application/json',
  },
});

console.log('Old key deleted successfully');
```

## BYOK Advantage

If you use Bring Your Own Key (BYOK) with OpenRouter, you can rotate your OpenRouter API keys without ever needing to rotate your provider keys. Your provider API keys are stored securely in OpenRouter and associated with your account, not with individual OpenRouter API keys. This separation simplifies compliance workflows while maintaining stable provider connections.

## Best Practices

- **Use descriptive key names**: Include rotation dates or version numbers for easy tracking (e.g., `Production Key - Rotated 2025-01`)
- **Monitor key usage**: Check the Activity page to verify traffic has migrated to the new key before deleting the old one
- **Set appropriate spending limits**: Apply spending limits on new keys to prevent unexpected costs
- **Document your rotation schedule**: Establish a regular rotation cadence (e.g., quarterly)
- **Test in staging first**: Always verify the rotation process in a staging environment before applying to production

## Related Documentation

- [Management API Keys](/docs/guides/overview/auth/provisioning-api-keys) - Complete key management API reference
- [API Authentication](/docs/api/reference/authentication) - Authentication methods overview
- [Infisical Integration](/docs/guides/community/infisical) - Automated key management with Infisical
