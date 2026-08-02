---
name: Authenticate to the Morning Consult API
description: Exchange Morning Consult Intelligence credentials for a JWT, send it on every request, and refresh it before it expires.
api: openapi/morning-consult-openapi-original.yml
operations:
  - postAuthToken
generated: '2026-08-01'
method: generated
source: openapi/morning-consult-openapi-original.yml + https://api.morningconsult.com/docs/#authentication
---

# Authenticate to the Morning Consult API

Every Morning Consult operation except the token exchange itself requires a JWT bearer
token. There is no API key. Base URL is `https://api.morningconsult.com/v1`.

## Prerequisites

- A Morning Consult Intelligence username and password. These are issued by the
  customer's Morning Consult Account Executive — there is no self-service signup.
- If the account signs in to MCI with SSO, a separate API password must be requested
  from the Account Executive; SSO cannot mint a token at this endpoint.

## Steps

1. **Exchange credentials.** Call `postAuthToken` (`POST /auth/token`) with HTTP Basic
   auth (`--user username:password`). No request body is required.
2. **Read the token pair.** The response carries:
   - `id_token` — the JWT to send as `Authorization: Bearer <id_token>`
   - `refresh_token` — exchange this for a new pair before expiry
   - `expires_in` — seconds until `id_token` expires (3600 in the published default)
   - `token_type` — always `Bearer`
3. **Send it.** Put `Authorization: Bearer <id_token>` on every other call.
4. **Refresh before expiry.** Call `postAuthToken` again, this time authenticating with
   the `RefreshToken` bearer scheme (`Authorization: Bearer <refresh_token>`), and
   replace both tokens with the new pair. Do not wait for a 401.

## Rules

- **Never re-exchange credentials per request.** The Auth bucket allows 20 requests per
  minute per username and one per second. Cache the JWT for its full lifetime and
  refresh once, near expiry.
- Refresh-token requests are limited to 30 per minute.
- A `400` with `errors: ["invalid credentials"]` means the username/password was
  rejected; `["invalid or expired refresh token"]` means the refresh window was missed —
  fall back to a full credential exchange.
- A `403` with `errors: ["missing necessary claims"]` on a later call is not an
  authentication failure: the token is valid but the subscription lacks that
  entitlement (typically bulk / Data Bridge access). Do not retry or re-authenticate.

See `authentication/morning-consult-authentication.yml` and
`conventions/morning-consult-conventions.yml`.
