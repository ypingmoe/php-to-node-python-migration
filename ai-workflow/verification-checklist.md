# Verification before merge

Use this on every change that a model helped write. A pull request names the model in the slot from `README.md`.

## Source integrity

- [ ] CVE numbers, fixed versions, and “exploited in the wild” claims link to a primary source I opened (PHP advisory, CISA KEV, or vendor notice)
- [ ] The diff contains no exploit steps, payloads, or attack traffic
- [ ] Assumptions introduced by the model are labeled or removed

## Contract

- [ ] Behavior matches `packages/contracts/API.md` for status codes and error `code`
- [ ] Both runtimes change together, or the PR explains why one runtime is untouched
- [ ] New routes are absent unless the contract was updated in the same PR

## Security baseline

- [ ] Authenticated routes reject a missing token
- [ ] Owner checks exist for content writes and upload reads
- [ ] Upload path enforces size and the media allowlist on the server
- [ ] Original filenames are not used as storage paths
- [ ] Secrets are not committed

## Tests and ops

- [ ] Contract or integration tests cover the success path and one auth failure
- [ ] Logs include the request id and the route template
- [ ] Health still reports database status
- [ ] I can explain the rollback (proxy rule or revert commit) in one sentence

## AI-specific

- [ ] A second read happened on auth, session, and upload diffs
- [ ] Generated tests were run, not only created
- [ ] Review-model comments are resolved or explicitly dismissed with a reason
