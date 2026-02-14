# Bring Your Own API Keys

OpenRouter supports both OpenRouter credits and the option to bring your own provider keys (BYOK).

When you use OpenRouter credits, your rate limits for each provider are managed by OpenRouter.

Using provider keys enables direct control over rate limits and costs via your provider account.

Your provider keys are securely encrypted and used for all requests routed through the specified provider.

Manage keys in your [account settings](https://openrouter.ai/settings/integrations).

The cost of using custom provider keys on OpenRouter is a percentage of what the same model/provider would cost normally on OpenRouter and will be deducted from your OpenRouter credits. This fee is waived for the first monthly BYOK requests per-month.

## Key Priority and Fallback

OpenRouter always prioritizes using your provider keys when available. By default, if your key encounters a rate limit or failure, OpenRouter will fall back to using shared OpenRouter credits.

You can configure individual keys with "Always use this key" to prevent any fallback to OpenRouter credits.

## BYOK with Provider Ordering

When you combine BYOK keys with provider ordering, OpenRouter always prioritizes BYOK endpoints first, regardless of where that provider appears in your specified order.

### Example JSON - Full BYOK

```json
{
  "provider": {
    "allow_fallbacks": true,
    "order": ["amazon-bedrock", "google-vertex", "anthropic"]
  }
}
```

### Example JSON - Partial BYOK

```json
{
  "provider": {
    "allow_fallbacks": true,
    "order": ["amazon-bedrock", "google-vertex"]
  }
}
```

## Azure API Keys

```json
{
  "model_slug": "the-openrouter-model-slug",
  "endpoint_url": "https://<resource>.services.ai.azure.com/deployments/<model-id>/chat/completions?api-version=<api-version>",
  "api_key": "your-azure-api-key",
  "model_id": "the-azure-model-id"
}
```

### Multiple Azure Deployments

```json
[
  {
    "model_slug": "mistralai/mistral-large",
    "endpoint_url": "https://example-project.openai.azure.com/openai/deployments/mistral-large/chat/completions?api-version=2024-08-01-preview",
    "api_key": "your-azure-api-key",
    "model_id": "mistral-large"
  },
  {
    "model_slug": "openai/gpt-5.2",
    "endpoint_url": "https://example-project.openai.azure.com/openai/deployments/gpt-5.2/chat/completions?api-version=2024-08-01-preview",
    "api_key": "your-azure-api-key",
    "model_id": "gpt-5.2"
  }
]
```

## AWS Bedrock API Keys

Simple format:

```
your-bedrock-api-key-here
```

### AWS Credentials Option

```json
{
  "accessKeyId": "your-aws-access-key-id",
  "secretAccessKey": "your-aws-secret-access-key",
  "region": "your-aws-region"
}
```

### Example IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": "*"
    }
  ]
}
```

## Google Vertex API Keys

```json
{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "your-private-key-id",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "your-service-account@your-project.iam.gserviceaccount.com",
  "client_id": "your-client-id",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/your-service-account@your-project.iam.gserviceaccount.com",
  "universe_domain": "googleapis.com",
  "region": "global"
}
```

### Example IAM Policy

```json
{
  "bindings": [
    {
      "role": "roles/aiplatform.user",
      "members": [
        "serviceAccount:your-service-account@your-project.iam.gserviceaccount.com"
      ]
    }
  ]
}
```

## Debugging BYOK Issues

Common HTTP status codes:

| Status Code | Meaning |
|-------------|---------|
| **400 Bad Request** | Invalid request format |
| **401 Unauthorized** | Invalid or revoked API key |
| **403 Forbidden** | Insufficient permissions |
| **429 Too Many Requests** | Rate limit exceeded |
| **500 Server Error** | Provider-side internal error |
