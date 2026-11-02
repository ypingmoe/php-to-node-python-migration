# Why owners leave PHP

**Audience:** founders and operators who have a PHP backend and want it on Node.js or Python  
**Purpose:** explain the business reason for the move

## What changed

In October 2026, two critical PHP vulnerabilities were published together as PHP2Shell:

- CVE-2026-109618
- CVE-2026-109619

On a typical PHP website they allow an unauthenticated attacker to run code on the server. Public reporting has described real exploitation, and CISA listed both issues. The primary write-up is the [CISA alert](https://www.cisa.gov/news-events/alerts/2026/10/21/critical-vulnerabilities-php-require-immediate-patching).

This note does not explain how the bugs work, and it is not a PHP patch guide.

## Who feels this

The people who write in are not PHP maintainers. They are:

- Founders whose product backend is a custom PHP app or a framework such as Laravel
- Companies running WordPress, Drupal, Magento, or another PHP CMS they do not want to keep operating
- Teams that already decided the next backend should be Node.js or Python

Another PHP release does not answer that decision. The work is to move the product.

## What the migration is

1. Name the routes customers actually use.
2. Rebuild those routes in Fastify (Node.js) or FastAPI (Python).
3. Switch one path at a time so the site stays up.
4. Remove PHP from the product when the new backend is serving it.

The order of work is the [migration playbook](../docs/migration-playbook.md). The example API shape is the [accounts, content, and upload contract](../packages/contracts/API.md).

## What I need from you

- What the product does
- Where the PHP code lives (custom app or a known CMS)
- Whether you want Node.js or Python
