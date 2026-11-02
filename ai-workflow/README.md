# AI workflow

This folder shows how I use current language models on a PHP-to-Node.js or PHP-to-Python migration. The standard is simple: models draft, people verify, CI decides.

**Positioning:** AI-accelerated delivery with a written review gate. Speed is useful only when the contract, the tests, and the security checklist still pass.

## Roles

| Role | What the model is good at | What stays with the engineer |
|---|---|---|
| Research | Turning the owner's PHP system into a first-draft migration plan | Checking that the plan matches the routes they actually use |
| Implementation | Drafting a handler, repository method, or migration checklist from an approved OpenAPI operation | Schema fit, error codes, auth checks, and making both runtimes match |
| Review | A second-pass checklist: parity, auth, uploads, logging, secrets | Deciding the checklist is complete and blocking merge on failures |

Use a stronger reasoning model for research and review. Use a fast coding model for mechanical drafts. Re-run review with a different model family than the one that wrote the draft when the change touches auth or uploads.

## Cutting-edge models

Update this table when you change tools. Do not treat the names as permanent.

| Slot | Model (update me) | Used for | Last reviewed |
|---|---|---|---|
| Research | @ypingmoe | Advisory summaries, ADR first drafts | 2026-11-03 |
| Implementation | @ypingmoe | Handlers, tests, OpenAPI fragments | 2026-11-03 |
| Review | @ypingmoe | Security and parity checklist | 2026-11-03 |

**Selection rules**

- Prefer a model with a long context window when the input is an OpenAPI file plus an ADR.
- Prefer a model that cites sources, then still open those sources.
- Record the model id in the pull request when a generated artifact is committed.
- Replace a model that invents CVE details or expands scope into offensive guidance.

## What models may draft

- Migration plans from the owner's PHP route list
- ADR prose from a decision I already made
- OpenAPI operation sketches that follow `packages/contracts/API.md`
- Test plans and migration checklists
- Review comments against a diff

## What models may not ship

- Exploit steps, payloads, proof-of-concept requests, or “lab” attack traffic
- Auth bypasses, broader upload allowlists, or disabled scan gates
- Uncited vulnerability claims
- A second runtime implementation copied blindly without contract tests

## Files

| File | Use |
|---|---|
| [prompts.md](prompts.md) | Templates |
| [verification-checklist.md](verification-checklist.md) | Gate before merge |
| [examples/content-platform-migration-checklist.md](examples/content-platform-migration-checklist.md) | Sample AI-assisted artifact |
