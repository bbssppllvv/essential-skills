# Snowflake Broadcast Integration

OpenRouter enables streaming traces directly to Snowflake for analytics, storage, and business intelligence purposes.

## Setup Process

### Step 1: Create Traces Table
Create the `OPENROUTER_TRACES` table in Snowflake using SQL specifications available in the OpenRouter dashboard configuration interface.

### Step 2: Generate Access Credentials
Obtain a Programmatic Access Token with `ACCOUNTADMIN` permissions through Snowflake's Settings > Authentication menu.

### Step 3: Enable Broadcast
Navigate to Settings > Observability in OpenRouter and activate the Broadcast feature.

### Step 4: Configure Snowflake Details
Enter the following information via the Snowflake edit interface:

- **Account**: Snowflake account identifier (format: `region/account-number`)
- **Token**: Your Programmatic Access Token
- **Database**: Target database (default: `SNOWFLAKE_LEARNING_DB`)
- **Schema**: Target schema (default: `PUBLIC`)
- **Table**: Table name (default: `OPENROUTER_TRACES`)
- **Warehouse**: Compute warehouse (default: `COMPUTE_WH`)

### Step 5: Test Connection
Verify configuration with the Test Connection button; settings save only upon successful validation.

### Step 6: Validate with Test Trace
Execute an API request through OpenRouter and query the Snowflake table to confirm trace receipt.

## Sample Queries

**Cost Analysis by Model**: Aggregates daily spending and token usage by model for 30-day periods.

**User Activity Analysis**: Tracks per-user metrics including trace counts, session counts, token usage, and average latency over 7 days.

**Error Analysis**: Retrieves failed requests with trace details, timestamps, and model information from the past hour.

**Provider Performance**: Compares providers by average duration and percentile response times.

**API Key Usage**: Summarizes token consumption and costs by API key across 30 days.

**VARIANT Column Access**: Demonstrates querying semi-structured data using Snowflake's casting syntax.

## Schema Structure

**Typed Columns** include trace identifiers, timestamps, model information, and billing metrics for efficient querying.

**VARIANT Columns** store less-frequent data: `ATTRIBUTES`, `INPUT`, `OUTPUT`, `METADATA`, and `MODEL_PARAMETERS`.

## Custom Metadata

Store application-specific data via the `trace` field object. Supported keys include `trace_id`, `trace_name`, `span_name`, and `generation_name`. Query custom fields using Snowflake's semi-structured syntax with the `METADATA` column.

## Privacy Mode

When enabled, prompt and completion content excludes from traces while preserving token counts, costs, timing, model data, and custom metadata.
