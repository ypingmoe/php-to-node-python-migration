# ADR-001: Reference stack and Node.js vs Python selection

- **Status:** Accepted
- **Date:** 2026-11-03

## Context

Clients arriving after PHP2Shell own a product with a PHP backend and want that product on Node.js or Python. A real engagement picks one of those runtimes.

The Node side needs a framework that can share a single OpenAPI contract with Python. Nest is a strong fit for large teams that want modular structure similar to a mature Laravel codebase. Fastify is a smaller, schema-first framework. FastAPI is the Python counterpart: explicit models, OpenAPI generation, and a production track record.

The reference product is an accounts, content, and upload API. It is I/O-bound. It does not require a machine-learning runtime.

## Decision

1. Ship the reference Node.js API with **Fastify** and the reference Python API with **FastAPI**.
2. Both implementations follow `packages/contracts` and the same Postgres schema.
3. On a client engagement, choose **one** primary runtime with this order of questions:
   - Which language does the team already operate in production?
   - Is the next product work mostly JSON APIs and realtime I/O (favor Node.js) or data processing, reporting, and scientific libraries (favor Python)?
   - Are there internal libraries that exist in only one language?
   - Who will be on call six months after the rewrite?
4. Recommend Nest only when the client explicitly wants a highly modular Node.js codebase and accepts the extra framework surface.
5. Treat PHP as the system being removed. The engagement is done when product traffic is served by Node.js or Python.

## Consequences

- The public repo demonstrates two production-shaped services instead of a single favored stack.
- Fastify and FastAPI stay easy to compare because both lean on declared schemas.
- A team that later standardizes on Nest can still reuse the OpenAPI contract, schema, and playbook.
- Maintaining two reference apps has a cost. Feature work lands in the contract first, then in both apps, so they do not drift.
- The assessment memo for a client names the chosen runtime in the first page, with the criteria above filled in from their team, not from this demo.
