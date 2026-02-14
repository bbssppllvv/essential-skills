# APIKeys - TypeScript SDK

## Overview

The APIKeys section provides methods for managing API keys within the OpenRouter TypeScript SDK. These endpoints handle creation, retrieval, updating, and deletion of API keys for authenticated users.

**Note:** "The TypeScript SDK and docs are currently in beta. Report issues on GitHub."

## Available Operations

- **list** - Retrieve all API keys
- **create** - Generate a new API key
- **update** - Modify an existing API key
- **delete** - Remove an API key
- **get** - Fetch a specific API key
- **getCurrentKeyMetadata** - Obtain current session's API key information

## Common Requirements

All operations require a provisioning key for authentication.

## Method Examples

### List API Keys
```typescript
const result = await openRouter.apiKeys.list();
```

### Create API Key
```typescript
const result = await openRouter.apiKeys.create({
  name: "My New API Key",
});
```

### Update API Key
```typescript
const result = await openRouter.apiKeys.update({
  hash: "f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943",
  requestBody: {},
});
```

### Delete API Key
```typescript
const result = await openRouter.apiKeys.delete({
  hash: "f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943",
});
```

### Get Specific API Key
```typescript
const result = await openRouter.apiKeys.get({
  hash: "f01d52606dc8f0a8303a7b5cc3fa07109c2e346cec7c0a16b40de462992ce943",
});
```

### Get Current Key Metadata
```typescript
const result = await openRouter.apiKeys.getCurrentKeyMetadata();
```

## Error Handling

Common error responses include:
- 400: Bad Request
- 401: Unauthorized
- 404: Not Found
- 429: Too Many Requests
- 500: Internal Server Error
