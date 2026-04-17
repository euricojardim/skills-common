---
name: ga4-analytics
description: Query Google Analytics 4 data through the analytics-mcp server (account/property discovery, custom dimensions & metrics, core reports, realtime reports, Google Ads links, annotations). Use when the user asks about GA4 metrics, traffic, events, users, conversions, realtime visitors, or anything backed by a GA4 property.
---

# GA4 Analytics via analytics-mcp

This skill wraps the [google-analytics-mcp](https://github.com/googleanalytics/google-analytics-mcp) server (`analytics-mcp` PyPI package). It exposes the GA4 Admin API and Data API (including realtime) as MCP tools.

## When to use

Invoke when the user asks anything requiring live GA4 data:
- "How many users on my site last week?"
- "What are the top events in the last 180 days?"
- "Who's on the site right now?"
- "What's my Google Ads link for property X?"
- "List my GA4 properties"

If the MCP server is not configured, guide the user through setup (see **Setup** below) instead of guessing data.

## Tools exposed by the server

Tool names appear as MCP tools. When this project's Docker compose is running, they are reachable at `http://localhost:8000/sse`; from the Claude Code side they appear as `mcp__<server>__<tool>`.

### Discovery
- **`get_account_summaries`** — no args. Lists all GA4 accounts + properties the auth principal can see. **Always call this first when the user hasn't given a `property_id`.**
- **`get_property_details(property_id)`** — returns timezone, currency, industry category, etc.
- **`get_custom_dimensions_and_metrics(property_id)`** — returns `{custom_dimensions, custom_metrics}`. Call before using any non-standard field in a report.
- **`list_property_annotations(property_id)`** — user-added notes on GA4 dates (launches, campaigns).
- **`list_google_ads_links(property_id)`** — linked Google Ads accounts.

### Reporting
- **`run_report(property_id, date_ranges, dimensions, metrics, dimension_filter?, metric_filter?, order_bys?, limit?, offset?, currency_code?, return_property_quota?)`** — core Data API report.
- **`run_realtime_report(property_id, dimensions, metrics, dimension_filter?, metric_filter?, order_bys?, limit?, offset?, return_property_quota?)`** — last 30 minutes of activity.

## Critical rules

1. **Field names are snake_case**, not camelCase. The public REST docs use `startDate`, `matchType`, `fieldName` — but this server uses protobuf, so pass `start_date`, `match_type`, `field_name`. Converting from REST docs without renaming is the #1 failure mode.
2. **`property_id`** accepts either a bare number (`123456789`) or `"properties/123456789"`.
3. **Realtime reports** can use only realtime standard metrics (no custom metrics). For custom dimensions, only user-scoped ones (`apiName` starting with `customUser:`) are allowed.
4. **Standard field catalogs**:
   - Core dimensions/metrics: https://developers.google.com/analytics/devguides/reporting/data/v1/api-schema
   - Realtime dimensions/metrics: https://developers.google.com/analytics/devguides/reporting/data/v1/realtime-api-schema
5. **Filter independence**: `dimension_filter` and `metric_filter` are AND-combined at the top level. You cannot express `(D1 AND M1) OR (D2 AND M2)` in one request — run two reports, or over-fetch and filter client-side.
6. **Pagination**: `limit` ≤ 250,000 per page; use `offset` for subsequent pages.

## Argument formats

### `date_ranges` — list of `DateRange`
```json
[{"start_date": "2025-01-01", "end_date": "2025-01-31", "name": "Jan2025"}]
```
Relative tokens accepted: `"today"`, `"yesterday"`, `"NdaysAgo"` (e.g., `"30daysAgo"`).

### `dimensions` / `metrics` — lists of strings
```json
{"dimensions": ["country", "deviceCategory"], "metrics": ["activeUsers", "sessions"]}
```

### `dimension_filter` — `FilterExpression` (protobuf, snake_case)

Simple string filter:
```json
{"filter": {"field_name": "eventName", "string_filter": {"match_type": "BEGINS_WITH", "value": "add"}}}
```

In-list:
```json
{"filter": {"field_name": "eventName", "in_list_filter": {"case_sensitive": true, "values": ["first_visit", "purchase", "add_to_cart"]}}}
```

AND / OR group:
```json
{"and_group": {"expressions": [<expr1>, <expr2>]}}
{"or_group":  {"expressions": [<expr1>, <expr2>]}}
```

NOT:
```json
{"not_expression": <expr>}
```

Empty-value:
```json
{"filter": {"field_name": "source", "empty_filter": {}}}
```

### `metric_filter` — same `FilterExpression` shape but numeric
```json
{"filter": {"field_name": "eventCount", "numeric_filter": {"operation": "GREATER_THAN", "value": {"int64_value": 10}}}}
```
Between:
```json
{"filter": {"field_name": "purchaseRevenue", "between_filter": {"from_value": {"double_value": 10.0}, "to_value": {"double_value": 25.0}}}}
```

### `order_bys` — list
```json
[{"dimension": {"dimension_name": "eventName", "order_type": "ALPHANUMERIC"}, "desc": false}]
[{"metric": {"metric_name": "eventCount"}, "desc": true}]
```
Dimensions/metrics in `order_bys` must also be in the request's `dimensions`/`metrics`.

## Standard workflow

For any user question about their GA4 data:

1. **Resolve the property.** If the user didn't give one, call `get_account_summaries` and ask which property (unless there's only one match).
2. **If custom fields may be needed**, call `get_custom_dimensions_and_metrics(property_id)`.
3. **Build the report.** Pick the smallest `dimensions` set that answers the question — every extra dimension multiplies row count.
4. **Call `run_report`** (or `run_realtime_report` for "right now" questions).
5. **Summarize the result** for the user. Don't dump the raw JSON unless asked; pull out the numbers that answer the question and format as a short table or sentence.

## Example: "top 10 countries by active users last 30 days for property 123"

```json
{
  "property_id": 123,
  "date_ranges": [{"start_date": "30daysAgo", "end_date": "yesterday"}],
  "dimensions": ["country"],
  "metrics": ["activeUsers"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}],
  "limit": 10
}
```

## Example: "which paid-google events drove the most purchases last quarter"

```json
{
  "property_id": 123,
  "date_ranges": [{"start_date": "2026-01-01", "end_date": "2026-03-31"}],
  "dimensions": ["eventName"],
  "metrics": ["eventCount", "totalRevenue"],
  "dimension_filter": {
    "and_group": {"expressions": [
      {"filter": {"field_name": "sourceMedium", "string_filter": {"match_type": "EXACT", "value": "google / cpc"}}},
      {"filter": {"field_name": "eventName", "in_list_filter": {"values": ["purchase", "add_to_cart"]}}}
    ]}
  },
  "order_bys": [{"metric": {"metric_name": "eventCount"}, "desc": true}]
}
```

## Example: "who's on the site now"

```json
{
  "property_id": 123,
  "dimensions": ["country", "deviceCategory"],
  "metrics": ["activeUsers"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}],
  "limit": 20
}
```
(Realtime = last 30 min; no `date_ranges`.)

## Setup (if the MCP server is not running)

The project at `/home/ejardim/LABS/mcp/ga4_mcp` contains a Docker compose setup:

```bash
cd /home/ejardim/LABS/mcp/ga4_mcp
cp .env.example .env   # edit GOOGLE_PROJECT_ID, GOOGLE_ADC_PATH, optional GA4_PROPERTY_ID
docker compose up -d analytics-mcp
docker compose run --rm tester   # smoke test
```

Requirements:
- Google Cloud project with **Google Analytics Admin API** and **Google Analytics Data API** enabled.
- Application Default Credentials at `GOOGLE_ADC_PATH` (user ADC or impersonated SA) with scope `https://www.googleapis.com/auth/analytics.readonly`.
- The principal must have at least Viewer on the target GA4 property.

To use from Claude Code, add to the MCP server list:
```json
{
  "mcpServers": {
    "ga4": {
      "type": "sse",
      "url": "http://localhost:8000/sse"
    }
  }
}
```

## Common pitfalls

- **`PermissionDenied`**: the auth principal doesn't have access to that GA4 property. Fix in GA4 Admin → Property access management.
- **`Field X is not a valid dimension`**: likely using REST docs camelCase (e.g., `sessionSource` vs the actual field name), or the field is a custom dimension — check `get_custom_dimensions_and_metrics` first.
- **Realtime report returns empty**: no active users in the last 30 min, or the dimension/metric isn't in the realtime schema.
- **Quota errors**: set `return_property_quota: true` to see remaining tokens; batch with `limit`/`offset` instead of fetching all rows.
- **Filter type mismatch**: string fields need `string_filter`; numeric fields need `numeric_filter`. Mixing them raises an `InvalidArgument`.
