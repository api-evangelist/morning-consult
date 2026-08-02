---
name: Compute a Morning Consult score
description: Discover an available score such as Net Favorability, bind it to an entity and optional components, and request it as a timeseries for an audience.
api: openapi/morning-consult-openapi-original.yml
operations:
  - getScores
  - getScoreDataSources
  - getEntities
  - postScores
generated: '2026-08-01'
method: generated
source: openapi/morning-consult-openapi-original.yml + https://api.morningconsult.com/docs/#guides
---

# Compute a Morning Consult score

A score is a named metric computed from the survey data — Net Favorability, Total
Awareness and similar — rather than the raw responses to one question.

## Steps

1. **Discover the scores.** Call `getScores` (`GET /scores`), optionally filtered by
   `data_source_id`. Each `Score` has an `id`, `label`, `description`, an optional
   `entity_type` and an optional `components[]` list.
2. **Check availability.** Call `getScoreDataSources`
   (`GET /scores/{score_id}/data_sources`) to confirm which data sources — and therefore
   which countries — that score can be computed in.
3. **Resolve the entity.** If the score carries an `entity_type` (e.g. brand), call
   `getEntities` (`GET /entities`) with `query` set to the brand name and keep the
   `entity_id`.
4. **Request the score.** Call `postScores` (`POST /scores`) with:
   - `data_source_id` (required)
   - `score_id` (required)
   - `country_code` (required)
   - `entity_id` — required when the score's `entity_type` is present
   - `component_ids[]` — required only when computing a component-based score
   - `min_date` / `max_date`, `aggregation.interval`, and an optional `audience`
5. **Read the result.** Each `ScoreDataPoint` has a `date`, the computed `score` value,
   and the weighted `total_n` behind it.

## Rules

- `country_code` is a top-level requirement on `postScores` — unlike `postResponses`,
  where the country is implied by the question.
- Omitting a required `entity_id` for an entity-typed score returns a `400`; a
  `score_id` outside the subscription returns a `404`.
- Same Data bucket as `postResponses`: 200 requests per minute, shared.
- For more than a handful of score/entity/audience combinations, use the bulk path
  instead — see `morning-consult-bulk-data-bridge.md`.

See `data-model/morning-consult-data-model.yml` and
`rate-limits/morning-consult-rate-limits.yml`.
