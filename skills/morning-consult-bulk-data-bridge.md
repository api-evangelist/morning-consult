---
name: Load Morning Consult data in bulk via Data Bridge
description: Queue an asynchronous bulk responses or bulk scores request, poll it to completion, and download the Parquet result for a data warehouse.
api: openapi/morning-consult-openapi-original.yml
operations:
  - postResponsesBulk
  - getResponsesBulkStatus
  - postScoresBulk
  - getScoresBulkStatus
generated: '2026-08-01'
method: generated
source: openapi/morning-consult-openapi-original.yml + https://api.morningconsult.com/docs/#guides
---

# Load Morning Consult data in bulk via Data Bridge

Data Bridge is the warehouse path: one call can request up to 100,000 trend combinations
and returns a Parquet file for Snowflake, BigQuery or Databricks. It is asynchronous —
submit, then poll. There is **no webhook or callback**; polling is the only completion
signal.

## Steps — bulk responses

1. **Resolve IDs first.** Use the Metadata operations to collect every `question_id` you
   need and the response IDs for your audiences (see
   `morning-consult-custom-audience.md`). Do this once and cache — the bulk endpoint is
   metered per *day*, so a failed submission is expensive.
2. **Submit.** Call `postResponsesBulk` (`POST /responses/bulk`) with:
   - `data_source_id` (required)
   - `country` (required, 2-character code)
   - `question_ids[]` (required)
   - `aggregation` (required) — a bare string: `day`, `week`, `month`, `quarter` or
     `year`. Note this is the `BulkAggregation` form and does **not** accept `all`.
   - `audiences` — a map of your own audience names to audience definitions
   - `min_date` / `max_date` — both or neither, 10-year maximum span
3. **Keep the `request_id`** from the response.
4. **Poll.** Call `getResponsesBulkStatus` (`GET /responses/bulk/{request_id}`) until the
   job reports completion, then retrieve the Parquet download. Jobs typically finish in
   under an hour.

## Steps — bulk scores

Identical shape, using `postScoresBulk` (`POST /scores/bulk`) and
`getScoresBulkStatus` (`GET /scores/bulk/{request_id}`). Instead of `question_ids[]`,
send `scores` — a map of caller-defined names to score selections, where each selection
names a `score_id` and optionally an `entity_id` and `component_ids[]`.

## Rules

- **The Data Bridge bucket is 100 requests per day**, not per minute. Batch aggressively;
  do not submit one job per question.
- Poll on `getResponsesBulkStatus` / `getScoresBulkStatus`, which sit in the **Metadata**
  bucket (200/minute) — but poll on an interval of tens of seconds, not tight-loop.
- A `403` with `missing necessary claims` means the subscription does not include bulk /
  Data Bridge access. This is an entitlement problem, not an auth problem — do not retry.
- Submitting the same body twice creates a **second** job: these are the only operations
  in the API that are not naturally idempotent, and Morning Consult publishes no
  idempotency key. De-duplicate on your side before submitting.
- The JWT expires after 3600s; a long poll loop must refresh it (see
  `morning-consult-authenticate.md`).

See `conventions/morning-consult-conventions.yml` and
`rate-limits/morning-consult-rate-limits.yml`.
