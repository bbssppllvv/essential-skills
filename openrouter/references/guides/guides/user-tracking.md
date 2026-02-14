# User Tracking Documentation

## Overview

OpenRouter provides a user tracking feature through an optional `user` parameter in API requests. This allows developers to identify their end-users and gain performance and analytics benefits.

## Core Functionality

The `user` parameter accepts any string identifier representing your application's end-user. As the documentation states, this could be "a user ID, email hash, session identifier, or any other stable identifier" used in your system.

## Primary Benefits

**Caching Optimization**: When you consistently use the same user identifier, OpenRouter can implement "sticky" caching that keeps individual users routed to the same provider, maintaining warm caches while allowing load distribution across different providers for separate users.

**Analytics Access**: User identifiers appear in the activity dashboard, data exports, and the generations API, enabling detailed usage breakdown by individual user.

## Implementation

The feature is straightforward to implement across multiple SDKs:

- **TypeScript SDK**: Include `user: 'user_12345'` in the request object
- **Python/OpenAI SDK**: Pass `user="user_12345"` to the `chat.completions.create()` method
- **TypeScript/OpenAI SDK**: Add `user: 'user_12345'` to completion parameters

## Best Practices

1. **Maintain consistency**: Use the same identifier format throughout your application
2. **Choose stable identifiers**: Avoid random strings that vary between requests
3. **Protect privacy**: Use internal IDs rather than personal information in identifiers
