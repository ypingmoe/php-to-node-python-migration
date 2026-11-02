# Migration playbook

Move a company PHP backend to Fastify or FastAPI one route group at a time. The job is finished when the product runs on Node.js or Python.

**Related:** [ADR-001](adr/0001-reference-stack-and-runtime-selection.md), [ADR-002](adr/0002-strangler-migration.md), [API catalog](../packages/contracts/API.md), [why owners leave PHP](../research/why-owners-leave-php.md).

## Phases

### 0. Scope the move

- List the PHP routes customers actually use.
- Name the founder or operator who can accept the new behavior.
- Pick Node.js or Python with ADR-001. Record the reason in the engagement note.
- Choose the first route to leave PHP.

**Exit:** the new runtime is chosen and the first route is named.

### 1. Contract

- Choose the Phase 1 operations from `packages/contracts/API.md`.
- Write OpenAPI for those operations only.
- Capture fixture responses the owner accepts.
- List data the new service will own (users, content, upload metadata).

**Exit:** contract reviewed by the person who will call the API.

### 2. Build the pilot

- Implement the auth slice, then content, then uploads.
- Keep PHP as the upstream for every path you have not finished.
- Store secrets in the environment. Ship structured logs and `/health`.

**Exit:** contract tests pass against the new service in staging. PHP still serves production traffic.

### 3. Cut over

- Change the proxy for one prefix at a time (`/v1/auth`, then `/v1/me` and `/v1/content`, then `/v1/uploads`).
- Use an observation window (define the hours in ADR-006 before production).
- Rollback is restoring that prefix to PHP.

**Exit:** Phase 1 prefixes stay on the new service through the window with health checks green.

### 4. Expand and leave PHP

- Repeat phases 1–3 for later endpoints (password reset, scan callback, admin).
- Move or archive remaining PHP data.
- Remove the PHP upstream after a dated rollback deadline.
- Record the date PHP stopped serving the product.

**Exit:** product traffic has a single application runtime. PHP is gone or limited to a documented internal job with an owner.

## Risks

| Risk | Mitigation |
|---|---|
| PHP stays on the public site for a long time | Each phase removes a customer route from PHP, and the overlap has an end date |
| Two sources of truth for users | Pilot uses an explicit id map or a separate store; document which |
| Upload route accepts types the old app rejected | Server-side allowlist from the contract; tests for `415` and `413` |
| Rollback loses new writes | List tables written only by the new API before cutover |
| AI draft widens scope | Verification checklist in `ai-workflow/` |
| Team runs two stacks longer than planned | Overlap is scheduled. Each phase has an exit |

## Acceptance criteria (engagement-level)

- [ ] Written runtime choice (Node.js or Python) with the ADR-001 questions answered
- [ ] Phase 1 operations match the contract in both staging tests and proxy config
- [ ] Auth, owner checks, and upload limits covered by tests
- [ ] `/health` fails when the database is down
- [ ] Request id visible from proxy to app log
- [ ] Staging rollback rehearsed and timed
- [ ] The plan names the new runtime and the first route leaving PHP
- [ ] Repository and handover docs contain no exploit material

## Assumption

The edge proxy can route by path prefix. If the PHP app rewrites every URL to a single front controller and the proxy cannot see the path, the first task is to expose a stable prefix or route on a header the proxy can match.
