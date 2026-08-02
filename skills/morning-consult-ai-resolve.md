---
name: Ask Morning Consult a natural-language question
description: Use the AI resolve endpoint to answer a standalone question against Morning Consult's syndicated public-opinion data.
api: openapi/morning-consult-openapi-original.yml
operations:
  - postAIResolve
generated: '2026-08-01'
method: generated
source: openapi/morning-consult-openapi-original.yml + https://api.morningconsult.com/docs/#ai
---

# Ask Morning Consult a natural-language question

`POST /ai/resolve` pulls from respondent interviews and produces a summary of the data
that answers a plain-language question — Morning Consult's own agentic surface over the
survey corpus.

## Steps

1. **Authenticate.** Send `Authorization: Bearer <id_token>`.
2. **Call `postAIResolve`** (`POST /ai/resolve`) with a body of `{"text": "<query>"}`.
   Published example: `{"text": "What do women think of Cheerios?"}`.
3. **Wait.** Responses typically take around 10 seconds, and upwards of 20 seconds for
   complex queries. Set client timeouts accordingly.

## Rules

- **The query must be standalone.** Conversation history is not preserved between calls,
  so each request must carry its own full context. Do not build a chat loop that relies
  on the server remembering the previous turn.
- `text` is required, 1–1000 characters, and must match `^[^\0]+$`.
- The AI bucket is **20 requests per rolling 5-minute window** — far tighter than the
  Metadata and Data buckets. Queue and throttle rather than fanning out.
- Answers are summaries derived from the survey corpus. When you need the numbers behind
  an answer, follow up with `postResponses` or `postScores` against the specific
  question or score — see `morning-consult-brand-trendline.md`.

See `rate-limits/morning-consult-rate-limits.yml` and
`agentic-access/morning-consult-agentic-access.yml`.
