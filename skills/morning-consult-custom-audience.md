---
name: Build a custom audience for a Morning Consult data request
description: Compose a boolean audience out of question and response IDs and apply it as a respondent filter on a responses or scores request.
api: openapi/morning-consult-openapi-original.yml
operations:
  - getDataSourceQuestions
  - getDataSourceQuestion
  - postResponses
  - postScores
generated: '2026-08-01'
method: generated
source: openapi/morning-consult-openapi-original.yml + https://api.morningconsult.com/docs/#guides
---

# Build a custom audience for a Morning Consult data request

In Morning Consult every question is both a thing you can measure **and** a thing you can
filter by. An audience is a recursive boolean expression over question/response IDs.

## Steps

1. **Find the filtering questions.** Call `getDataSourceQuestions`
   (`GET /data_sources/{data_source_id}/countries/{country_code}/questions`) with a
   `query` or `category_id` to locate the demographic or attitudinal questions you want
   to filter on (gender, age, region, category usage, ...).
2. **Get the response options.** Call `getDataSourceQuestion`
   (`GET /data_sources/{data_source_id}/countries/{country_code}/questions/{question_id}`)
   and read `responses[]` — each entry has an `id` and a `label`. The `id` values are
   what an audience matches on.
3. **Compose the audience.** The `audience` field accepts either:
   - a **match**: `{"question_id": "<uuid>", "response_ids": ["<uuid>", ...]}` — matches
     respondents who chose any one of those options; or
   - a boolean node: `{"and": [...]}`, `{"or": [...]}`, `{"not": [...]}`, each holding
     further audience definitions. These nest arbitrarily.
4. **Attach it.** Send `audience` in the body of `postResponses` (`POST /responses`) or
   `postScores` (`POST /scores`) alongside `data_source_id` and the rest of the request.
5. **Vary it.** To compare audiences, re-issue the same request with a different
   `audience` value; to remove a condition, drop that element from the `and`/`or` array.

## Rules

- **Maximum 20 conditions per request**, counting every nested condition inside `and`,
  `or` and `not`. Exceeding it returns a `400` naming the field.
- Response IDs are only valid for their own question — mixing them across questions
  produces a `400`, not an empty result.
- Question and response IDs are stable forever. Resolve them once and cache them; do not
  re-run lookup calls per data request, since the Metadata bucket is 200/minute.
- The audience filters *which respondents are counted*; it does not change the shape of
  the response. `total_n` will shrink as the audience narrows — check it before drawing
  conclusions from a small base.

See `data-model/morning-consult-data-model.yml` and
`conventions/morning-consult-conventions.yml`.
