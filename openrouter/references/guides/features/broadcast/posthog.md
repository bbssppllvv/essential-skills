# Broadcast to PostHog

[PostHog](https://posthog.com) is an open-source product analytics platform that helps you understand user behavior. With PostHog's LLM analytics, you can track and analyze your AI application usage.

## Setup

### Step 1: Get your PostHog project API key

Navigate to your PostHog project settings and locate your Project API Key. It starts with `phc_`.

### Step 2: Enable Broadcast in OpenRouter

Go to [Settings > Observability](https://openrouter.ai/settings/observability) and toggle **Enable Broadcast**.

### Step 3: Configure PostHog

Click the edit icon next to **PostHog** and enter:

- **API Key**: Your PostHog project API key (starts with `phc_...`)
- **Endpoint** (optional): Defaults to `https://us.i.posthog.com`. For EU region, use `https://eu.i.posthog.com`.

### Step 4: Test and save

Click **Test Connection** to verify the setup. The configuration only saves if the test passes.

### Step 5: Send a test trace

Make an API request through OpenRouter and view the LLM analytics in your PostHog dashboard.

## Custom Metadata

PostHog receives LLM analytics events with custom metadata included as event properties. You can attach additional context to your traces by including a `trace` field in your request.

### Supported metadata keys

| Key | PostHog Mapping | Description |
|-----|-----------------|-------------|
| `trace_id` | Event property | Custom trace identifier for grouping related events |
| `trace_name` | Event property | Custom name for the trace |
| `generation_name` | Event property | Name for the LLM generation event |

### Additional field mappings

- The `user` field maps to PostHog's `$ai_user` property
- The `session_id` field maps to PostHog's `$ai_session_id` property
- Custom metadata keys from `trace` are included as event properties
- Token usage, costs, and model performance are automatically tracked

### Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{ "role": "user", "content": "Recommend a product..." }],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_name": "Product Recommendations",
    "generation_name": "Generate Recommendation",
    "feature": "shopping-assistant",
    "ab_test_group": "variant_b"
  }
}
```

## Privacy Mode

When Privacy Mode is enabled for this destination, the `$ai_input` and `$ai_output_choices` properties are excluded from events. All other analytics data -- token usage, costs, model information, and custom metadata -- is still sent normally.
