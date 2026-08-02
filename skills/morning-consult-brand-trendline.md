---
name: Pull a brand trendline from Morning Consult
description: Resolve a data source, country and question, then retrieve aggregated survey responses over a date range as a timeseries.
api: openapi/morning-consult-openapi-original.yml
operations:
  - postAuthToken
  - getDataSources
  - getEntities
  - getDataSourceQuestions
  - postResponses
generated: '2026-08-01'
method: generated
source: openapi/morning-consult-openapi-original.yml + https://api.morningconsult.com/docs/#guides
---

# Pull a brand trendline from Morning Consult

The single most common flow: find the question that measures what you care about about a
brand, then ask for its responses over time.

## Steps

1. **Authenticate.** See `morning-consult-authenticate.md`. Send
   `Authorization: Bearer <id_token>` on every call below.
2. **Choose a data source.** Call `getDataSources` (`GET /data_sources`). Pick the
   `id` you want to work with — for classic syndicated tracking this is the Daily
   Tracker. Read `countries[]` on that data source and pick a valid `code`.
3. **Find the entity (optional).** Call `getEntities` (`GET /entities`) with `query`
   set to the brand name to get its `entity_id`. Use it to filter step 4.
4. **Find the question.** Call `getDataSourceQuestions`
   (`GET /data_sources/{data_source_id}/countries/{country_code}/questions`), filtering
   with `query`, `category_id` and/or `entity_id`. Page with `page_size` (max 100) and
   `pagination_token`. Keep the `id` of the question you want — question IDs never
   change, so cache them permanently.
5. **Request the data.** Call `postResponses` (`POST /responses`) with a body of:
   - `data_source_id` (required)
   - `question_id` (required)
   - `min_date` / `max_date` in `YYYY-MM-DD` — both or neither, max 10-year span
   - `aggregation.interval` — one of `day`, `week`, `month`, `quarter`, `year`, `all`
6. **Read the result.** Each `TimeseriesDataPoint` has a `date` (start of its interval),
   a weighted `total_n`, and a `responses[]` array of `DataPoint` objects giving the
   `percent` for each response option.

## Rules

- Both `min_date` and `max_date` must be supplied together or omitted together. Omitting
  both returns the last 10 years.
- The `week` interval starts on Monday; `all` collapses the whole range into one point.
- `POST /responses` is in the **Data** bucket: 200 requests per minute. Watch
  `X-Rate-Limit-Remaining` and back off using `X-Rate-Limit-Reset` (UTC epoch ms).
- Errors are `{code, status, errors[]}` — not RFC 9457. The `errors[]` strings name the
  offending field, e.g. `invalid value "not-a-uuid" for field "data_source_id"`.
- Do **not** use `/surveys/syndicated/*` — that family sunsets 2026-10-01.

See `conventions/morning-consult-conventions.yml`, `errors/morning-consult-problem-types.yml`,
`rate-limits/morning-consult-rate-limits.yml`.
