# ClickHouse Integration Documentation

OpenRouter enables streaming traces directly to ClickHouse for real-time analytics and custom dashboards.

## Setup Process

### Step 1: Create Traces Table
Generate the `OPENROUTER_TRACES` table using SQL instructions provided in the OpenRouter dashboard's broadcast configuration.

### Step 2: Grant Database Permissions
Execute this command to authorize your ClickHouse user:
```sql
GRANT CREATE TABLE ON your_database.* TO your_database_user;
```

### Step 3: Enable Broadcast Feature
Navigate to Settings > Observability and activate the Broadcast toggle.

### Step 4: Configure Connection Details
Provide these ClickHouse parameters:
- **Host**: HTTP endpoint (e.g., `https://clickhouse.example.com:8123`)
- **Database**: Target database name (default: `default`)
- **Table**: Table identifier (default: `OPENROUTER_TRACES`)
- **Username**: Authentication user (defaults to `default`)
- **Password**: Authentication credential

For ClickHouse Cloud instances, the host typically follows this pattern: `https://{instance}.{region}.clickhouse.cloud:8443`

### Step 5: Validate Configuration
Test the connection to confirm successful setup before saving.

### Step 6: Verify with Sample Data
Send a test request through OpenRouter and query the table to confirm data ingestion.

## Query Examples

**Cost Analysis by Model**:
Aggregates spending and token usage across models over 30 days, filtered for successful generation spans.

**User Activity Analysis**:
Tracks unique traces, sessions, tokens, costs, and response times per user over 7 days.

**Error Diagnostics**:
Displays failed requests with timestamps, models, finish reasons, and message content from the past hour.

**Provider Performance Comparison**:
Calculates latency metrics across providers, comparing average, median, and 95th percentile response times.

**Usage by API Key**:
Summarizes request counts, expenses, and token consumption per key over 30 days.

## Working with JSON Data

ClickHouse stores variable-structure data as JSON strings. Use `JSONExtract` functions to access nested fields:

```sql
SELECT JSONExtractString(METADATA, 'custom_field') FROM OPENROUTER_TRACES
```

## Schema Organization

**Typed Columns**: Frequently-queried fields like trace IDs, timestamps, model names, and token metrics.

**JSON Columns**: Less common or variable data stored as strings (ATTRIBUTES, INPUT, OUTPUT, METADATA, MODEL_PARAMETERS).

## Custom Metadata Support

The `trace` object parameter supports custom key-value pairs mapped to the METADATA JSON column:

| Metadata Key | Mapping | Purpose |
|---|---|---|
| `trace_id` | TRACE_ID / METADATA JSON | Custom grouping identifier |
| `trace_name` | METADATA JSON | Trace label |
| `span_name` | METADATA JSON | Span identifier |
| `generation_name` | METADATA JSON | LLM generation label |

Query custom metadata using ClickHouse JSON functions:
```sql
SELECT JSONExtractString(METADATA, 'team') FROM OPENROUTER_TRACES
```

## Privacy Considerations

Privacy Mode excludes prompt and completion text from traces while preserving token counts, expenses, timing data, model information, and custom metadata.
