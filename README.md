# Move your PHP backend to Node.js or Python

I help people who own a product or run a company take a PHP backend off the product and replace it with **Node.js** or **Python**.

[![Node.js](https://img.shields.io/badge/Node.js-Fastify-339933)](docs/adr/0001-reference-stack-and-runtime-selection.md)
[![Python](https://img.shields.io/badge/Python-FastAPI-3776AB)](docs/adr/0001-reference-stack-and-runtime-selection.md)
[![Contract](https://img.shields.io/badge/Contract-OpenAPI-6BA539)](packages/contracts/API.md)
[![Data](https://img.shields.io/badge/Postgres-16-336791)](docs/adr)

**GitHub:** [@ypingmoe](https://github.com/ypingmoe)

## Who this is for

- You have your own project, or you run the company that owns it
- The backend is PHP: a custom app, Laravel, Symfony, WordPress, or another PHP CMS
- [PHP2Shell](research/why-owners-leave-php.md) (CVE-2026-109618 and CVE-2026-109619) made you want that backend on Node.js or Python

The work is the move to Node.js or Python.

## What you hire me for

| Engagement | What you leave with | Time |
|---|---|---|
| Migration plan | A map of the PHP backend and a written plan onto Node.js or Python | About 1 week |
| Pilot | One real feature of your product running on the new backend, on the same site | About 2–4 weeks |
| Full move | The rest of the backend moved, and PHP removed from the product | Scoped after the pilot |

You choose one new runtime. Node.js uses Fastify. Python uses FastAPI.

## How the move works

The site stays on its current address. PHP keeps serving only the routes that have not moved yet. Each finished route is switched to the new service. When the last product route has moved, PHP is removed.

```text
                 ┌─────────────┐
  customers ────►│  your site  │
                 └──────┬──────┘
                        │ one path at a time
            ┌───────────┴───────────┐
            ▼                       ▼
     ┌─────────────┐        ┌──────────────┐
     │  PHP        │        │ Node.js or   │
     │  (leaving)  │        │ Python       │
     └─────────────┘        └──────┬───────┘
                                   │
                            ┌──────▼───────┐
                            │  Postgres    │
                            └──────────────┘
```

1. Write down what one PHP route does.
2. Build that route in Fastify or FastAPI.
3. Send only that path to the new service.
4. Check it against the contract.
5. Repeat until customers are talking only to Node.js or Python.
6. Remove PHP from the product.

Details: [ADR-002](docs/adr/0002-strangler-migration.md) and the [migration playbook](docs/migration-playbook.md).

## What is in this repo

| Path | Purpose |
|---|---|
| [research/why-owners-leave-php.md](research/why-owners-leave-php.md) | Why owners with a PHP backend are moving |
| [packages/contracts/API.md](packages/contracts/API.md) | Example product API: accounts, content, uploads |
| [docs/migration-playbook.md](docs/migration-playbook.md) | The order of work on a client project |
| [docs/adr](docs/adr) | Node.js vs Python, and how the cutover works |
| [ai-workflow](ai-workflow) | How I use current AI models, and what I check before shipping |

The example product is accounts, content, and file upload. The same contract is the target for both Node.js and Python.

| Concern | Node.js | Python |
|---|---|---|
| HTTP | Fastify | FastAPI |
| Contract | One shared API catalog | Same catalog |
| Data | Postgres | Same schema |
| Auth | Short-lived access token, rotating refresh cookie | Same |
| Uploads | Size cap, allowed types, private storage | Same |

The runtime choice for a real company is in [ADR-001](docs/adr/0001-reference-stack-and-runtime-selection.md).

## How I use AI

I use current models to draft migration notes, API handlers, and tests. I ship a change after I have reviewed it and the tests pass. The method is in [ai-workflow](ai-workflow).

## Start a conversation

Tell me what the product does, that the backend is PHP, and whether you want Node.js or Python.

## Out of scope

- Exploit code and attack demos
- PHP maintenance and PHP upgrades
- Testing systems you do not own

## Keywords

`php2shell` `php to node` `php to python` `php migration` `nodejs` `python` `fastify` `fastapi` `backend` `founders`
