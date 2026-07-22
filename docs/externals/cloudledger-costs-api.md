---
id: ext/cloudledger-costs-api
kind: external
subkind: peer-system
system: cloudwatchr
name: CloudLedger Costs API
tech: CloudLedger Inventory API — Costs domain (Go/Fiber, OpenAPI-first)
owner: platform-eng            # owned by the CloudLedger team, not us
tags: [finops, cost, cross-repo, upstream]
relations: []
links:
  repo: https://github.com/dewrich/cloudledger
  container: https://github.com/dewrich/cloudledger/blob/main/docs/containers/inventory-api.md
  openapi: https://github.com/dewrich/cloudledger/blob/main/api/openapi/costs/v1/api-spec.yml
---

# CloudLedger Costs API — external (peer system, cross-repo)

This node represents **another repository's** container inside CloudWatchr's boundary: the
**Costs domain** of the [CloudLedger Inventory API](https://github.com/dewrich/cloudledger/blob/main/docs/containers/inventory-api.md).
It is external *to CloudWatchr* even though it is a first-party system — it lives in, is
versioned in, and is owned by [`dewrich/cloudledger`](https://github.com/dewrich/cloudledger).

## Why it is modelled as an external

The two systems are **separate repos with a one-way dependency**: CloudWatchr reads CloudLedger,
never the reverse. Modelling CloudLedger's Costs API as an `external` node here (rather than
inlining its containers) keeps each repo's C4 graph self-contained while still recording the
edge that crosses the repo boundary. The reciprocal edge — "CloudLedger has a downstream
consumer named CloudWatchr" — is recorded on
[CloudLedger's side](https://github.com/dewrich/cloudledger/blob/main/docs/context/index.md#downstream-consumers).

## What we call

Read-only, over HTTPS, with a Cognito bearer token scoped `costs:view`
(see [AWS Cognito](./aws-cognito.md)):

| Operation | Endpoint | Used by rule kind |
|---|---|---|
| Spend by service | `GET /costs/rollups/by-service` | `service_threshold` |
| Daily cost rows | `GET /costs/daily?account_id=&from=&to=` | `account_budget`, `spike` |

Source of truth for the request/response shapes is CloudLedger's OpenAPI document:
[`api/openapi/costs/v1/api-spec.yml`](https://github.com/dewrich/cloudledger/blob/main/api/openapi/costs/v1/api-spec.yml).
Responses carry `collected_at`, which the [Alerter](../containers/alerter.md) records for
freshness.

## Coupling & versioning

- **Contract:** the CloudLedger Costs OpenAPI spec (`v1`). Breaking changes there are breaking
  changes for CloudWatchr — the `X-Api-Version` header pins the major we build against.
- **No DB coupling.** CloudWatchr has no access to CloudLedger's `inventory-db`; the API is the
  only sanctioned read path, per CloudLedger's own
  [contract docs](https://github.com/dewrich/cloudledger/blob/main/docs/contracts/inventory-db.md).
