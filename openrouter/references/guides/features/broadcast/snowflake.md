# Broadcast to Snowflake

[Snowflake](https://www.snowflake.com) is a cloud data warehouse platform. OpenRouter can stream traces directly to your Snowflake database for custom analytics, long-term storage, and business intelligence.

## Setup

### Step 1: Create the traces table

Set up the `OPENROUTER_TRACES` table in your Snowflake database. You can find the exact SQL in the OpenRouter dashboard when configuring the destination.

### Step 2: Generate access credentials

Generate a Programmatic Access Token with `ACCOUNTADMIN` permissions in the Snowflake UI under Settings > Authentication.

### Step 3: Enable Broadcast in OpenRouter

Go to [Settings > Observability](https://openrouter.ai/settings/observability) and toggle **Enable Broadcast**.

### Step 4: Configure Snowflake

Click the edit icon next to **Snowflake** and enter:

- **Account Identifier**: Your Snowflake account identifier (e.g., `eac52885.us-east-1`). You can find this in your Snowflake instance URL.
- **Token**: Your Programmatic Access Token
- **Database**: Target database name (default: `SNOWFLAKE_LEARNING_DB`)
- **Schema**: Target schema (default: `PUBLIC`)
- **Table**: Table name (default: `OPENROUTER_TRACES`)
- **Warehouse**: Compute warehouse name

### Step 5: Test and save

Click **Test Connection** to verify the setup. The configuration only saves if the test passes.

### Step 6: Send a test trace

Make an API request through OpenRouter and query your Snowflake table to confirm the trace was received.

## Table Schema

The schema uses typed columns for commonly-queried fields and VARIANT columns for variable-structure data:

**Typed columns** include identifiers, timestamps, model information, and metrics for efficient filtering.

**VARIANT columns** store flexible semi-structured data:

- `ATTRIBUTES` - Request attributes
- `INPUT` - Input messages and content
- `OUTPUT` - Output responses
- `METADATA` - Custom metadata from the trace field
- `MODEL_PARAMETERS` - Model configuration parameters

## Custom Metadata

All custom metadata keys from `trace` are stored in the `METADATA` VARIANT column for flexible querying.

### Example

```json
{
  "model": "openai/gpt-4o",
  "messages": [{"role": "user", "content": "Forecast next quarter revenue..."}],
  "user": "user_12345",
  "session_id": "session_abc",
  "trace": {
    "trace_name": "Revenue Forecasting",
    "generation_name": "Generate Forecast",
    "department": "finance",
    "quarter": "Q2-2026",
    "model_version": "v3"
  }
}
```

## Example Queries

### Cost analysis by model

```sql
SELECT DATE_TRUNC('day', TIMESTAMP) as day, MODEL, SUM(TOTAL_COST) as total_cost,
SUM(TOTAL_TOKENS) as total_tokens, COUNT(*) as request_count
FROM OPENROUTER_TRACES WHERE TIMESTAMP >= DATEADD(day, -30, CURRENT_TIMESTAMP())
AND STATUS = 'ok' AND SPAN_TYPE = 'GENERATION'
GROUP BY day, MODEL ORDER BY day DESC, total_cost DESC;
```

### User activity analysis

```sql
SELECT USER_ID, COUNT(DISTINCT TRACE_ID) as trace_count,
COUNT(DISTINCT SESSION_ID) as session_count, SUM(TOTAL_TOKENS) as total_tokens,
SUM(TOTAL_COST) as total_cost, AVG(DURATION_MS) as avg_duration_ms
FROM OPENROUTER_TRACES WHERE TIMESTAMP >= DATEADD(day, -7, CURRENT_TIMESTAMP())
AND SPAN_TYPE = 'GENERATION' GROUP BY USER_ID ORDER BY total_cost DESC;
```

### Error analysis

```sql
SELECT TRACE_ID, TIMESTAMP, MODEL, LEVEL, FINISH_REASON,
METADATA as user_metadata, INPUT, OUTPUT FROM OPENROUTER_TRACES
WHERE STATUS = 'error' AND TIMESTAMP >= DATEADD(hour, -1, CURRENT_TIMESTAMP())
ORDER BY TIMESTAMP DESC;
```

### Provider performance comparison

```sql
SELECT PROVIDER_NAME, MODEL, AVG(DURATION_MS) as avg_duration_ms,
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY DURATION_MS) as p50_duration_ms,
PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY DURATION_MS) as p95_duration_ms,
COUNT(*) as request_count FROM OPENROUTER_TRACES
WHERE TIMESTAMP >= DATEADD(day, -7, CURRENT_TIMESTAMP()) AND STATUS = 'ok'
AND SPAN_TYPE = 'GENERATION' GROUP BY PROVIDER_NAME, MODEL
HAVING request_count >= 10 ORDER BY avg_duration_ms;
```

### Usage by API key

```sql
SELECT API_KEY_NAME, COUNT(DISTINCT TRACE_ID) as trace_count,
SUM(TOTAL_COST) as total_cost, SUM(PROMPT_TOKENS) as prompt_tokens,
SUM(COMPLETION_TOKENS) as completion_tokens FROM OPENROUTER_TRACES
WHERE TIMESTAMP >= DATEADD(day, -30, CURRENT_TIMESTAMP())
AND SPAN_TYPE = 'GENERATION' GROUP BY API_KEY_NAME ORDER BY total_cost DESC;
```

### Accessing VARIANT columns

```sql
SELECT TRACE_ID, METADATA:custom_field::STRING as custom_value,
ATTRIBUTES:"gen_ai.request.model"::STRING as requested_model
FROM OPENROUTER_TRACES WHERE METADATA:custom_field IS NOT NULL;
```

### Parsing input messages

```sql
SELECT TRACE_ID, INPUT:messages[0]:role::STRING as first_message_role,
INPUT:messages[0]:content::STRING as first_message_content
FROM OPENROUTER_TRACES WHERE SPAN_TYPE = 'GENERATION';
```

### Querying custom metadata

```sql
SELECT TRACE_ID, METADATA:department::STRING as department,
METADATA:quarter::STRING as quarter, METADATA:model_version::STRING as model_version,
TOTAL_COST, TOTAL_TOKENS FROM OPENROUTER_TRACES
WHERE METADATA:department IS NOT NULL AND SPAN_TYPE = 'GENERATION'
ORDER BY TIMESTAMP DESC;
```

## Privacy Mode

When Privacy Mode is enabled for this destination, prompt and completion content is excluded from traces. All other trace data -- token usage, costs, timing, model information, and custom metadata -- is still sent normally.
