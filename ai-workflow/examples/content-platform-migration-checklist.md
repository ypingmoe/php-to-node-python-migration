# Migration checklist — Content Platform Phase 1

**Status:** example artifact  
**How this was produced:** drafted from `packages/contracts/API.md`.  
**New runtime for this example:** FastAPI behind the existing edge. Fastify would use the same boxes.  
**Legacy:** PHP still serves any path not listed as cut over.

This is a delivery checklist. It does not describe how to attack PHP.

## Before any route moves

- [ ] Name the PHP routes this pilot will take off PHP, using the migration plan
- [ ] Capture fixtures for the Phase 1 responses the product owner accepts as correct
- [ ] Publish `packages/contracts` to both app teams
- [ ] Add proxy rules in “PHP” mode for `/v1/*` so the new upstream exists but receives no traffic
- [ ] Confirm request ids are generated at the edge

## Auth slice

- [ ] `POST /v1/auth/register` — persist user, argon2id hash, return access token, set refresh cookie
- [ ] `POST /v1/auth/login` — identical error body for unknown user and bad password
- [ ] `POST /v1/auth/refresh` — rotate refresh token; reuse of the previous token revokes the chain
- [ ] `POST /v1/auth/logout` — revoke and clear cookie
- [ ] Contract tests green on FastAPI (and Fastify if both are in scope)
- [ ] Cut over only `/v1/auth/*` at the proxy
- [ ] Watch auth failure rate for the observation window in ADR-006
- [ ] Rollback: point `/v1/auth/*` back at PHP

**Assumption:** the PHP login and the new login share one user table only after a mapped id exists. Until that map is done, the pilot uses a separate user store and is not the production login.

## Account and content slice

- [ ] `GET /v1/me` and `PATCH /v1/me`
- [ ] `GET /v1/content` with cursor and draft visibility
- [ ] `POST /v1/content`, `GET /v1/content/{id}`, `PATCH`, `DELETE` with owner checks
- [ ] Problem responses use the catalog `code` values
- [ ] Cut over `/v1/me` and `/v1/content` together, because the UI needs both
- [ ] Rollback: restore those two prefixes to PHP

## Upload slice

- [ ] `POST /v1/uploads` rejects oversize bodies with `413` / `payload_too_large`
- [ ] Non-allowlisted types fail with `415` / `unsupported_media_type` after sniffing
- [ ] Object key is server-generated
- [ ] `GET /v1/uploads/{id}/content` returns bytes only when `scan_status=clean`
- [ ] A content `POST` with a pending upload fails with `409` or `422` as specified once OpenAPI locks the code
- [ ] Cut over `/v1/uploads` after the content slice has been stable
- [ ] Rollback: restore `/v1/uploads` to PHP and stop linking new upload ids

## Exit criteria for Phase 1

- [ ] Every Phase 1 operation has a passing contract test on the chosen runtime
- [ ] Proxy config for the pilot is in version control
- [ ] A rehearsed rollback was executed in staging
- [ ] Runtime choice (ADR-001) and the first route leaving PHP are attached to the engagement notes
- [ ] No exploit material entered the repo during AI-assisted drafts
