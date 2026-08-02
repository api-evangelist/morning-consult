# Morning Consult

Morning Consult is a decision intelligence company that fields more than 30,000 online
surveys every day across 45+ countries, turning continuous consumer interviewing into
brand, reputation, economic and category tracking data.

Its flagship product, Morning Consult Intelligence (MCI), exposes that data
programmatically through the **Morning Consult API** — a versioned REST API at
`https://api.morningconsult.com/v1` with a published OpenAPI 3.0.3 document and a
custom-built documentation site.

- Website: https://morningconsult.com/
- API product page: https://morningconsult.com/products/intelligence/api-aggregate-daily-data
- API documentation: https://api.morningconsult.com/docs/
- OpenAPI: https://api.morningconsult.com/openapi.yaml

## API at a glance

| | |
|---|---|
| Base URL | `https://api.morningconsult.com/v1` |
| Spec | OpenAPI 3.0.3, 24 paths / 26 operations, 42 schemas |
| Tags | Authentication, Lookup, Data, AI |
| Auth | HTTP Basic credential exchange at `POST /auth/token` → JWT (3600s) + refresh token |
| Pagination | Opaque base64 `pagination_token` cursor, `page_size` max 100 |
| Errors | Custom `{code, status, errors[]}` JSON envelope (not RFC 9457) |
| Rate limits | Bucketed — Auth 20/min, Metadata 200/min, Data 200/min, Data Bridge 100/day, AI 20 per 5 min |
| Bulk | Submit-then-poll Data Bridge jobs returning Parquet for Snowflake / BigQuery / Databricks |
| Deprecation | The `/v1/surveys/syndicated/*` family is `deprecated: true` in-spec and **sunsets 2026-10-01** |
| Events | None — no webhooks, no AsyncAPI, no streaming surface |
| SDKs | None — the docs publish cURL / Python / Node.js / Go request examples instead |

## Artifacts in this repo

| Directory | What it holds |
|---|---|
| `openapi/` | The verbatim OpenAPI 3.0.3 spec harvested from `api.morningconsult.com/openapi.yaml` |
| `json-schema/` | The 42 component schemas as a JSON Schema 2020-12 document |
| `overlays/` | API Evangelist annotations over the spec (rate-limit buckets, sunset dates, async shape) |
| `authentication/` | The Basic → JWT → refresh auth profile |
| `conventions/` | Pagination, date ranges, audiences, aggregation, error envelope, async, versioning |
| `errors/` | The error catalog derived from spec responses and the published error table |
| `rate-limits/` | The full published bucket table plus `X-Rate-Limit-*` signaling |
| `lifecycle/` | Versioning, the dated deprecation policy and the full migration map |
| `data-model/` | The survey-measurement entity graph derived from the spec |
| `examples/` | The provider's published request/response examples and guide walkthroughs |
| `conformance/` | Standards conformance and the published compliance program |
| `security/` | Domain security probe, vulnerability disclosure policy, trust center |
| `well-known/` | The `/.well-known/*` probe record (all negative) |
| `packages/` | Package registry findings — no first-party API client SDK exists |
| `mcp/` | A derived *candidate* MCP tool surface; Morning Consult publishes no MCP server |
| `skills/` | Six generated Agent Skills grounded in real operationIds |
| `arazzo/` | Three native Arazzo 1.0.1 workflows |
| `agentic-access/` | Generated `x-agentic-access` classifications for all 26 operations |
| `llms/` | Morning Consult's own `llms.txt`, saved verbatim |

## Notable gaps

- No public API status page.
- No dated changelog — the Migration Guide is the only published record of change.
- No `/.well-known/security.txt`, OIDC discovery, api-catalog or A2A agent card.
- No MCP server and no A2A agent card.
- No idempotency contract; the bulk submit operations are the only non-idempotent calls.
- `429` is documented and enforced but never declared as a response in the OpenAPI.
- Deprecation is dated and marked in-spec, but no RFC 8594 `Sunset`/`Deprecation` headers.
