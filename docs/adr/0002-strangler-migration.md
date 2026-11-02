# ADR-002: Strangler migration at the edge

- **Status:** Accepted
- **Date:** 2026-11-03

## Context

Replacing a PHP backend in one release concentrates product risk: auth, data, uploads, and customer traffic all change on the same day. PHP is the legacy backend this engagement removes.

The accounts and content demo has a small number of resources and a clear edge. That matches the usual client case: a proxy or web server already sits in front of PHP.

## Decision

Migrate with a **strangler** at the reverse proxy.

1. Freeze the behavior of a route by writing it into the OpenAPI contract and capturing fixtures from agreed responses.
2. Implement that route on the new service. PHP continues to serve every other route.
3. Point only the finished path at the new service.
4. Run contract tests and watch logs for an agreed observation window.
5. Repeat until the route map sends product traffic only to Node.js or Python.
6. Decommission the PHP vhost after backups and a written rollback date that has passed.

Rules while both stacks are live:

- The proxy is the only place that chooses PHP versus the new API.
- Each request carries one request id from the edge into whichever backend handles it.
- A pilot route writes its new data to Postgres owned by the new API. Cross-writes into PHP tables require an explicit note in the playbook.
- Rollback of a route is a proxy-config change that restores the previous upstream.

## Consequences

- Users keep one hostname. Clients do not need a flag day.
- Routes still on PHP are leftover product traffic. Each phase shrinks that set until PHP is gone.
- Delivery is incremental: auth and content can ship before the full upload pipeline.
- The team operates two runtimes until the last route moves. The playbook treats that overlap as a budgeted cost.
- Rollback is real only if the proxy change is small and rehearsed. ADR-006 will record the exact observation signals.
- Routes that share in-memory PHP session state need the auth bridge in ADR-003 before they move.
