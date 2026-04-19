---
name: bigquery
description: Query Google BigQuery through the official bigquery.googleapis.com MCP server (dataset/table discovery, schema inspection, SQL execution). Use when the user asks to count, aggregate, join, explore, or otherwise interrogate data stored in BigQuery — including the GA4→BigQuery export. Always discover the schema before writing SQL; never guess column names.
---

# BigQuery via the official MCP server

This skill wraps Google's first-party **BigQuery MCP server** (`https://bigquery.googleapis.com/mcp`, HTTP transport, OAuth 2.0 + IAM). It exposes dataset/table metadata and SQL execution as MCP tools.

## When to use

Invoke for any question that requires reading data from BigQuery:
- "How many sessions did we have last week?" (GA4 export)
- "What's the schema of `analytics_123456789.events_*`?"
- "List the datasets in my project."
- "Top 10 pages by pageviews yesterday."
- "Join orders with customers and aggregate by country."

If the MCP server isn't configured, walk the user through **Setup** instead of fabricating results.

## Tools exposed by the server

The MCP tool names appear in Claude Code as `mcp__<server>__<tool>`. The core tools are:

- **`list_dataset_ids(project_id)`** — enumerate datasets in a GCP project. Always call first if the user didn't specify a dataset.
- **`list_table_ids(project_id, dataset_id)`** — list tables in a dataset.
- **`get_dataset_info(project_id, dataset_id)`** — dataset metadata (location, labels, default expiration).
- **`get_table_info(project_id, dataset_id, table_id)`** — table schema (column names, types, modes), row count, size, partitioning, clustering. **Call this before writing any SQL that references a table you haven't already inspected this session.**
- **`execute_sql(sql, project_id?, ...)`** — run a SQL query and return rows. Use **Standard SQL** (GoogleSQL) dialect.

The server may expose additional tools depending on the preview version the user has enabled — if a tool the user asks for isn't listed here, query the server's `tools/list` or check the [reference](https://docs.cloud.google.com/bigquery/docs/reference/mcp).

## Critical rules

1. **Always discover before querying.** Call `list_dataset_ids` → `list_table_ids` → `get_table_info` to resolve the real column names and types. Guessing a column name is the #1 failure mode.
2. **Cost control.** Before running an unbounded `SELECT` on a large table, add a partition filter (`WHERE _TABLE_SUFFIX BETWEEN ...` or `WHERE event_date BETWEEN ...`) and a `LIMIT`. Mention the estimated/actual bytes processed when you can, so the user can catch runaway queries.
3. **Use `_TABLE_SUFFIX` for sharded tables** like `events_YYYYMMDD` rather than scanning all shards.
4. **Fully qualify tables** as `` `project.dataset.table` `` in SQL unless a default project/dataset is set.
5. **GoogleSQL dialect only** — do NOT emit legacy SQL. Use backticks for identifiers, `STRUCT`/`ARRAY` handling, and `UNNEST` for repeated fields.
6. **Summarize results**; don't dump raw JSON unless asked. Pull out the numbers that answer the question and format as a short table or sentence.

## Standard workflow

For any data question:

1. **Resolve the project.** If the user didn't specify one, ask — or use the default configured on the server.
2. **Discover the dataset.** If unknown, call `list_dataset_ids`. If GA4 is expected, look for `analytics_<property_id>`.
3. **Discover the table + schema.** Call `list_table_ids` and then `get_table_info` for each table you'll touch.
4. **Draft SQL** that answers the question with the smallest scan possible (partition filter, minimal columns, `LIMIT`).
5. **Execute** via `execute_sql`.
6. **Summarize** the result for the user.

## GA4 → BigQuery export (common case for this user)

The user has a GA4 stream piped into BigQuery. The export creates these shards inside the `analytics_<property_id>` dataset:

| Table pattern | Contents |
|---|---|
| `events_YYYYMMDD` | Daily finalized events (available ~next day). |
| `events_intraday_YYYYMMDD` | Today's events, appended in near-real-time, replaced when the daily table is finalized. |
| `pseudonymous_users_YYYYMMDD` | Daily snapshot of user properties keyed by `user_pseudo_id`. |
| `users_YYYYMMDD` | Daily snapshot keyed by User-ID (if set). |

### Key columns on `events_*`

- `event_date` (STRING, `YYYYMMDD`), `event_timestamp` (INT64 µs), `event_name` (STRING).
- `event_params` — `ARRAY<STRUCT<key STRING, value STRUCT<string_value, int_value, float_value, double_value>>>`. Extract with `UNNEST`.
- `user_id`, `user_pseudo_id`, `user_properties` (same nested shape as `event_params`).
- `device`, `geo`, `app_info`, `traffic_source`, `collected_traffic_source`, `session_traffic_source_last_click` — each a STRUCT.
- `ecommerce` STRUCT + `items` ARRAY for purchase/commerce events.
- `privacy_info`, `platform`, `stream_id`.

### Extracting event parameters

GA4 doesn't expose scalar columns for custom params — you pull them out of `event_params` with `UNNEST`:

```sql
SELECT
  event_date,
  event_name,
  (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'page_location') AS page_location,
  (SELECT value.int_value    FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS ga_session_id
FROM `my-proj.analytics_123456789.events_*`
WHERE _TABLE_SUFFIX BETWEEN '20260401' AND '20260418'
  AND event_name = 'page_view'
LIMIT 100
```

### Sessions

A GA4 session is the tuple `(user_pseudo_id, event_params.ga_session_id)`. Count distinct sessions like:

```sql
SELECT COUNT(DISTINCT CONCAT(
  user_pseudo_id,
  CAST((SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS STRING)
)) AS sessions
FROM `my-proj.analytics_123456789.events_*`
WHERE _TABLE_SUFFIX BETWEEN '20260401' AND '20260407'
```

### Including intraday

To get "up to now" numbers, UNION the intraday shard:

```sql
FROM (
  SELECT * FROM `my-proj.analytics_123456789.events_*`
   WHERE _TABLE_SUFFIX BETWEEN '20260401' AND '20260418'
  UNION ALL
  SELECT * FROM `my-proj.analytics_123456789.events_intraday_*`
   WHERE _TABLE_SUFFIX = '20260419'
)
```

### Common GA4 queries

**Top pages by pageviews, last 7 days:**
```sql
SELECT
  (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'page_location') AS page,
  COUNT(*) AS pageviews
FROM `my-proj.analytics_123456789.events_*`
WHERE _TABLE_SUFFIX BETWEEN FORMAT_DATE('%Y%m%d', DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY))
                        AND FORMAT_DATE('%Y%m%d', DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND event_name = 'page_view'
GROUP BY page
ORDER BY pageviews DESC
LIMIT 10
```

**Purchases + revenue by source/medium, last 30 days:**
```sql
SELECT
  traffic_source.source AS source,
  traffic_source.medium AS medium,
  COUNT(*) AS purchases,
  SUM(ecommerce.purchase_revenue) AS revenue
FROM `my-proj.analytics_123456789.events_*`
WHERE _TABLE_SUFFIX BETWEEN FORMAT_DATE('%Y%m%d', DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY))
                        AND FORMAT_DATE('%Y%m%d', DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND event_name = 'purchase'
GROUP BY source, medium
ORDER BY revenue DESC
LIMIT 20
```

**Active users by country, yesterday:**
```sql
SELECT
  geo.country AS country,
  COUNT(DISTINCT user_pseudo_id) AS active_users
FROM `my-proj.analytics_123456789.events_*`
WHERE _TABLE_SUFFIX = FORMAT_DATE('%Y%m%d', DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
GROUP BY country
ORDER BY active_users DESC
LIMIT 20
```

### GA4 vs. BigQuery — expect small deltas

Numbers from BigQuery will not match the GA4 UI exactly because:
- GA4 UI applies thresholding, sampling for unsampled reports, and (HyperLogLog++) cardinality estimates.
- BigQuery export is raw — you choose the session/user definition.
- Late-arriving hits can shift daily totals for ~72h after event time.

Mention this explicitly if the user compares the two.

## Non-GA4 workflows

The skill is not GA4-only — treat it as a general BigQuery client:

1. `list_dataset_ids(project_id)` to see what's there.
2. `get_table_info` on relevant tables to read the schema.
3. Write GoogleSQL, executed via `execute_sql`.

When joining across datasets/projects, fully qualify every table and confirm the user has cross-project access.

## Setup (if the MCP server is not registered)

**Endpoint:** `https://bigquery.googleapis.com/mcp` (HTTP)

**Auth:** OAuth 2.0 + IAM — no API keys. The calling principal needs:
- `roles/mcp.toolUser`
- `roles/bigquery.jobUser` (to run queries)
- `roles/bigquery.dataViewer` on each dataset to read (or broader at project level)

OAuth scope: `https://www.googleapis.com/auth/bigquery`

**Register in Claude Code** (`~/.claude.json` or project `.mcp.json`):
```json
{
  "mcpServers": {
    "bigquery": {
      "type": "http",
      "url": "https://bigquery.googleapis.com/mcp"
    }
  }
}
```

Claude Code will prompt the OAuth flow on first use; alternatively, run the server against local Application Default Credentials (`gcloud auth application-default login`) through the MCP Toolbox for Databases if the remote server isn't available in the user's region.

References:
- [Use the BigQuery MCP server](https://docs.cloud.google.com/bigquery/docs/use-bigquery-mcp)
- [BigQuery MCP reference](https://docs.cloud.google.com/bigquery/docs/reference/mcp)
- [GA4 BigQuery Export schema](https://support.google.com/analytics/answer/7029846)

## Common pitfalls

- **`Not found: Table ...`** — the project/dataset/table names are case-sensitive and must be backtick-quoted in SQL.
- **`Query exceeded resource limits`** — add a partition filter, narrow columns, or use `LIMIT`. For GA4, always filter by `_TABLE_SUFFIX`.
- **`PermissionDenied`** — the principal lacks `bigquery.jobUser` or `dataViewer` on the target. Check IAM.
- **Empty `event_params` extracts** — the subquery returns NULL when the key is absent; that's expected. Test with a known key like `page_location` first.
- **Doubled rows after UNION with intraday** — make sure the intraday date range doesn't overlap with the finalized date range.
- **Comparing with GA4 UI** — numbers won't match exactly; see the note above.
