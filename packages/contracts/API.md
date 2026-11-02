# Content Platform API catalog

**Contract version:** 0.1.0  
**Base URL:** `/v1`  
**Demo domain:** accounts, content items, and file uploads  
**Consumers:** the Fastify app, the FastAPI app, and contract tests

**Assumption:** Phase 1 is the pilot slice a client would fund first. Later endpoints show the rest of a credible product without pretending they already exist in code.

## Resources

| Resource | Meaning |
|---|---|
| Session | Login, refresh, and logout for a user account |
| User | The authenticated account and profile |
| Content item | A title, body, and publication state owned by a user |
| Upload | A stored object that may be attached to a content item |
| Health | Process and dependency status for the proxy |

## Auth model

- `POST /v1/auth/register` and `POST /v1/auth/login` are public.
- All other business routes require `Authorization: Bearer <access_token>`.
- Access tokens are JWTs. **Assumption:** 15-minute lifetime.
- Refresh tokens are opaque, stored hashed on the server, and set as an `HttpOnly`, `Secure`, `SameSite=Lax` cookie named `refresh_token`. **Assumption:** 14-day lifetime, rotated on every refresh.
- Logout revokes the current refresh token.
- The legacy PHP session cookie is **not** part of this contract. Coexistence belongs in ADR-003 as a proxy-side adapter.
- Passwords are stored with a slow hash (argon2id). The API never returns a password hash.

### Roles (Phase 1)

| Role | Can |
|---|---|
| `user` | Manage own profile, own drafts, own uploads |
| `user` | Read published content from anyone |

Admin moderation is a later phase.

## Error model

Errors use `application/problem+json` (RFC 9457).

| Field | Required | Notes |
|---|---|---|
| `type` | yes | URI under `https://api.example.test/problems/` |
| `title` | yes | Stable short label |
| `status` | yes | HTTP status |
| `code` | yes | Machine code, snake_case |
| `detail` | yes | Safe for API clients |
| `instance` | yes | Request path |
| `request_id` | yes | Edge request id |
| `errors` | no | Field list for validation: `{ "field", "message" }` |

| HTTP | `code` | When |
|---|---|---|
| 400 | `bad_request` | Malformed JSON |
| 401 | `unauthorized` | Missing or invalid access token |
| 403 | `forbidden` | Authenticated, wrong owner |
| 404 | `not_found` | Unknown id |
| 409 | `conflict` | Duplicate email, or upload already finalized |
| 413 | `payload_too_large` | Body or file over the limit |
| 415 | `unsupported_media_type` | Type outside the allowlist |
| 422 | `validation_error` | Schema failed |
| 503 | `dependency_unavailable` | Database unavailable |

## Upload constraints (defensive)

| Rule | Phase 1 value |
|---|---|
| Max size | 10 MiB |
| Allowlist | `image/jpeg`, `image/png`, `image/webp`, `application/pdf` |
| Type check | Server-side sniff. Client `Content-Type` is not trusted |
| Storage key | Generated id. Original filename is metadata only |
| Location | Object store or disk path outside any public web root |
| Visibility | Bytes are served only through an authenticated API route |
| Gate | A content item may reference an upload only when `scan_status` is `clean` |
| Scan states | `pending`, `clean`, `rejected` |
| Rejected types | Archives, HTML, SVG, and anything executable are outside the allowlist |

**Assumption:** Phase 1 records `scan_status=pending` and a worker interface. A real scanner is an integration, not a fake “always clean” in production. The demo may mark a fixture `clean` only in tests.

## Phase 1 — pilot

These endpoints are enough to show auth parity, CRUD, and a safe upload path.

### Health

#### `GET /health`

Unauthenticated. Used by the proxy. Lives outside `/v1` so it stays stable.

Response `200`:

| Field | Type | Notes |
|---|---|---|
| `status` | string | `ok` or `degraded` |
| `service` | string | `node-api` or `python-api` |
| `version` | string | Build version |
| `checks.database` | string | `up` or `down` |

### Auth

#### `POST /v1/auth/register`

Request:

| Field | Type | Notes |
|---|---|---|
| `email` | string | Lowercased |
| `password` | string | Min 12 characters |
| `display_name` | string | 1–80 characters |

Response `201`: `User` body, plus access token JSON and `Set-Cookie: refresh_token`.

| Field | Type |
|---|---|
| `access_token` | string |
| `token_type` | `Bearer` |
| `expires_in` | number |
| `user` | User |

#### `POST /v1/auth/login`

Request: `email`, `password`.  
Response `200`: same token body as register.  
Failure `401` with `code=unauthorized`. Same detail for unknown email and bad password.

#### `POST /v1/auth/refresh`

Cookie: `refresh_token`.  
Response `200`: new access token body. Rotates the cookie.  
Failure `401` if the cookie is missing, reused, or revoked.

#### `POST /v1/auth/logout`

Requires bearer token. Revokes the refresh token and clears the cookie.  
Response `204`.

### Current user

#### `GET /v1/me`

Response `200` User:

| Field | Type |
|---|---|
| `id` | uuid |
| `email` | string |
| `display_name` | string |
| `created_at` | date-time |

#### `PATCH /v1/me`

Request: optional `display_name`.  
Response `200`: User.

### Content

#### `GET /v1/content`

Query: `status` (`published` or `draft`, optional), `limit` (default 20, max 100), `cursor` (optional).  
Response `200`:

| Field | Type |
|---|---|
| `items` | Content[] |
| `next_cursor` | string or null |

Drafts are returned only when they belong to the caller.

#### `POST /v1/content`

Request:

| Field | Type | Notes |
|---|---|---|
| `title` | string | 1–140 |
| `body` | string | Max 50_000 characters |
| `status` | string | `draft` or `published` |
| `upload_id` | uuid or null | Must be owned by the caller and `clean` |

Response `201`: Content.

#### `GET /v1/content/{content_id}`

Published items are world-readable to any authenticated user. Drafts are owner-only.  
Response `200`: Content.

#### `PATCH /v1/content/{content_id}`

Owner only. Same fields as create, all optional.  
Response `200`: Content.

#### `DELETE /v1/content/{content_id}`

Owner only. Response `204`.

**Content object**

| Field | Type |
|---|---|
| `id` | uuid |
| `author_id` | uuid |
| `title` | string |
| `body` | string |
| `status` | `draft` or `published` |
| `upload_id` | uuid or null |
| `created_at` | date-time |
| `updated_at` | date-time |

### Uploads

#### `POST /v1/uploads`

`multipart/form-data` with a single part named `file`.  
Response `201`:

| Field | Type |
|---|---|
| `id` | uuid |
| `media_type` | string |
| `byte_size` | number |
| `scan_status` | `pending` / `clean` / `rejected` |
| `created_at` | date-time |

#### `GET /v1/uploads/{upload_id}`

Owner only. Metadata, not bytes. Response `200`: the upload object.

#### `GET /v1/uploads/{upload_id}/content`

Owner only, and only when `scan_status=clean`.  
Response `200` with the stored bytes and `Content-Type` set from the sniffed type.  
`409` with `code=conflict` when the object is still `pending` or `rejected`.

## Later phases

Not in the first pilot. Listed so the contract has a direction.

| Phase | Endpoint | Purpose |
|---|---|---|
| 2 | `POST /v1/auth/password-reset/request` | Start a reset; always `202` |
| 2 | `POST /v1/auth/password-reset/confirm` | Consume a one-time token |
| 2 | `GET /v1/auth/sessions` | List refresh sessions |
| 2 | `DELETE /v1/auth/sessions/{session_id}` | Revoke one session |
| 3 | `POST /v1/uploads/{upload_id}/scan-callback` | Worker reports `clean` or `rejected` (internal auth) |
| 3 | `GET /v1/content/{content_id}` as public | Optional unauthenticated read of published items |
| 4 | `GET /v1/admin/content` | Moderator queue |
| 4 | `POST /v1/admin/content/{content_id}/takedown` | Hide a published item |

Internal scan callbacks use a separate `Authorization: Bearer <worker_token>` and are not browser routes.
