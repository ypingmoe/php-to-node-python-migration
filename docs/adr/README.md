# Architecture Decision Records

Decisions for the reference migration and for how a client engagement chooses a path.

| ADR | Title | Status |
|---|---|---|
| [0001](0001-reference-stack-and-runtime-selection.md) | Reference stack and Node.js vs Python selection | Accepted |
| [0002](0002-strangler-migration.md) | Strangler migration at the edge | Accepted |
| 0003 | Auth and session parity with legacy PHP | Proposed |
| 0004 | Safe file upload design | Proposed |
| 0005 | Observability baseline | Proposed |
| 0006 | Cutover and rollback | Proposed |
| 0007 | AI-assisted development policy | Proposed |

## 0003 — Auth and session parity with legacy PHP

PHP apps usually authenticate with a server-side session cookie. The new APIs authenticate with a short-lived bearer access token and a rotating refresh cookie. This ADR will define the bridge used during coexistence: which routes still accept the legacy session, how identity is mapped to the new user id, and when the legacy cookie stops being honored. The goal is a single account that works on both sides of the proxy until cutover, without copying PHP session files into the new service.

## 0004 — Safe file upload design

Uploads are a normal product feature and a common place for operational risk. This ADR will require an allowlist of media types, a hard size limit, a generated storage key, metadata-only retention of the original filename, storage outside any public document root, and a scan-status gate so content is not linked until the object is marked clean. Detection uses server-side type checks. The client-supplied `Content-Type` is only a hint.

## 0005 — Observability baseline

Operators need to see a route that has moved. This ADR will require structured request logs (method, route template, status, duration, request id), a `/health` check that covers process and database, and a small set of metrics: request count, latency, auth failures, and upload rejects. The legacy and new services must emit the same request id when the proxy forwards a call, so a failed pilot can be traced across both stacks.

## 0006 — Cutover and rollback

A route is cut over by changing the edge map, not by deleting PHP. This ADR will define the observation window, the health signals that keep a route on the new service, and the single-step rollback (restore the previous proxy rule). Data written only to the new database during a pilot must be listed before rollback so it is either migrated back or explicitly accepted as new-system data.

## 0007 — AI-assisted development policy

Models may draft research notes, OpenAPI fragments, handler sketches, test lists, and review checklists. A change may ship only after a person has checked it against the contract, the tests, and the security checklist in `ai-workflow/`. Models may not introduce exploit material, weaken auth, or expand upload types. Generated text that cites a vulnerability must link to a primary advisory the author has opened.
