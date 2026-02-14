# Guardrails - TypeScript SDK

## Overview

Documentation for the Guardrails endpoints in the OpenRouter TypeScript SDK. All operations require a provisioning API key.

## Available Operations

- **list** - List all guardrails
- **create** - Create a new guardrail
- **get** - Retrieve a guardrail by ID
- **update** - Modify an existing guardrail
- **delete** - Remove a guardrail
- **listKeyAssignments** - View all API key assignments
- **listMemberAssignments** - View all member assignments
- **listGuardrailKeyAssignments** - View key assignments for specific guardrail
- **bulkAssignKeys** - Assign multiple keys to guardrail
- **listGuardrailMemberAssignments** - View member assignments for specific guardrail
- **bulkAssignMembers** - Assign multiple members to guardrail
- **bulkUnassignKeys** - Remove multiple keys from guardrail
- **bulkUnassignMembers** - Remove multiple members from guardrail

## Core Methods

### list
Retrieves all guardrails for the authenticated user.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.list();
```

**Possible errors:** 401 Unauthorized, 500 Internal Server Error

### create
Establishes a new guardrail with specified configuration.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.create({
  name: "My New Guardrail",
});
```

**Possible errors:** 400 Bad Request, 401 Unauthorized, 500 Internal Server Error

### get
Fetches a single guardrail using its identifier.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.get({
  id: "550e8400-e29b-41d4-a716-446655440000",
});
```

**Possible errors:** 401 Unauthorized, 404 Not Found, 500 Internal Server Error

### update
Modifies properties of an existing guardrail.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.update({
  id: "550e8400-e29b-41d4-a716-446655440000",
  requestBody: {},
});
```

**Possible errors:** 400 Bad Request, 401 Unauthorized, 404 Not Found, 500 Internal Server Error

### delete
Removes a guardrail from the system.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.delete({
  id: "550e8400-e29b-41d4-a716-446655440000",
});
```

**Possible errors:** 401 Unauthorized, 404 Not Found, 500 Internal Server Error

## Assignment Methods

### listKeyAssignments
Lists all API key guardrail assignments.

### listMemberAssignments
Lists all organization member guardrail assignments.

### listGuardrailKeyAssignments
Lists API key assignments for a specific guardrail.

### bulkAssignKeys
Assigns multiple API keys to a guardrail.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.bulkAssignKeys({
  id: "550e8400-e29b-41d4-a716-446655440000",
  requestBody: {
    keyHashes: ["c56454edb818d6b14bc0d61c46025f1450b0f4012d12304ab40aacb519fcbc93"],
  },
});
```

### listGuardrailMemberAssignments
Lists organization member assignments for a specific guardrail.

### bulkAssignMembers
Assigns multiple organization members to a guardrail.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.bulkAssignMembers({
  id: "550e8400-e29b-41d4-a716-446655440000",
  requestBody: {
    memberUserIds: ["user_abc123", "user_def456"],
  },
});
```

### bulkUnassignKeys
Removes multiple API keys from a guardrail.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.bulkUnassignKeys({
  id: "550e8400-e29b-41d4-a716-446655440000",
  requestBody: {
    keyHashes: ["c56454edb818d6b14bc0d61c46025f1450b0f4012d12304ab40aacb519fcbc93"],
  },
});
```

### bulkUnassignMembers
Removes multiple organization members from a guardrail.

**Basic usage:**
```typescript
const result = await openRouter.guardrails.bulkUnassignMembers({
  id: "550e8400-e29b-41d4-a716-446655440000",
  requestBody: {
    memberUserIds: ["user_abc123", "user_def456"],
  },
});
```

## Notes

- The TypeScript SDK and documentation are in beta; report issues on GitHub
- All operations require a provisioning API key for authentication
- Standalone function versions available for tree-shaking optimization
