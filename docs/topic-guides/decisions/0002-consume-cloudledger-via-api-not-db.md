# 0002 — Consume CloudLedger via its API, not its database

## Context

CloudWatchr needs AWS spend to evaluate budget rules. That data already exists in
[CloudLedger](https://github.com/dewrich/cloudledger), which aggregates it into a Postgres
system-of-record and exposes a read-only [Costs domain API](https://github.com/dewrich/cloudledger/blob/main/docs/containers/inventory-api.md).
Two ways to get at it: read CloudLedger's database directly, or call its API.

CloudLedger's own [contract docs](https://github.com/dewrich/cloudledger/blob/main/docs/contracts/inventory-db.md)
are explicit that "the API is the only sanctioned read path" and that a second system reading the
DB directly would force promoting the store to a shared namespace. We are that second system.

## Decision

CloudWatchr consumes CloudLedger **exclusively through its Costs domain API** over HTTPS, with a
Cognito bearer token scoped `costs:view`. We do not connect to CloudLedger's database, replica or
otherwise. The dependency is modelled as an [external node](../../externals/cloudledger-costs-api.md)
and recorded as a cross-repo `reads` relation in frontmatter.

## Consequences

- **Loose coupling.** We depend on CloudLedger's versioned OpenAPI contract (`v1`,
  `X-Api-Version`), not its schema. CloudLedger can refactor its DB freely.
- **We inherit the API's freshness and auth model** — responses carry `collected_at`; every call
  needs a valid token. The Alerter surfaces both.
- **A one-way, documented boundary.** The edge crosses repos once, in one direction, and both
  repos record it (CloudWatchr as an external it reads; CloudLedger as a downstream consumer).
- **Trade-off:** an extra network hop and API dependency versus direct SQL. Acceptable — spend is
  evaluated every 15 minutes, not per-request.

## Status

Accepted
