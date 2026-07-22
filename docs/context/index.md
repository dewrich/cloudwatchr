---
id: cloudwatchr
kind: system
name: CloudWatchr
system: cloudwatchr
owner: observability-eng
tags: [finops, alerting, monitoring, batch]
relations:
  - to: cloudledger
    kind: reads
    desc: Reads aggregated AWS spend from CloudLedger's Costs domain API to evaluate budget rules
    tech: HTTPS / REST (costs:view) — cross-repo, see externals/cloudledger-costs-api.md
    repo: https://github.com/dewrich/cloudledger
---

# CloudWatchr — system (C4 L1 Context)

CloudWatchr is the bounded context for **cloud-spend alerting**. It does not collect or store
cost data; it **reads** the spend that [CloudLedger](https://github.com/dewrich/cloudledger)
aggregates, evaluates it against user-defined budget and threshold rules, and fires alerts to
the on-call channels when spend runs hot.

## Containers

- **[Alerter](../containers/alerter.md)** — a scheduled Go service that pulls cost rollups from
  the CloudLedger Costs API, evaluates every active rule, and records fired/cleared alert state.
- **[Notifier](../containers/notifier.md)** — an always-on Go service that turns fired alerts
  into deliveries (Slack, PagerDuty, email) with de-duplication and escalation.

## What the boundary talks to

- **Contract:** [`cloudwatchr/alert-state-db`](../contracts/alert-state-db.md) — the Postgres
  store for alert **rules** and fired-alert **state**. The Alerter writes it; the Notifier reads it.
- **Upstream external (cross-repo):**
  [CloudLedger Costs API](../externals/cloudledger-costs-api.md) — the read-only source of spend.
- **Identity external:** [AWS Cognito](../externals/aws-cognito.md) — CloudWatchr obtains a
  Cognito token with `costs:view` so the Alerter can call CloudLedger's API.
- **Notification externals:** [Slack](../externals/slack.md) and
  [PagerDuty](../externals/pagerduty.md) — where alerts land.

## Related systems

CloudWatchr is one half of a two-repo pair. The dependency points **one way**:

| System | Repo | Role | This system's link to it |
|---|---|---|---|
| **CloudLedger** | [dewrich/cloudledger](https://github.com/dewrich/cloudledger) | Cost & inventory system-of-record (upstream) | We **read** its [Costs domain API](https://github.com/dewrich/cloudledger/blob/main/docs/containers/inventory-api.md); modelled locally as [`ext/cloudledger-costs-api`](../externals/cloudledger-costs-api.md) |
| **CloudWatchr** | [dewrich/cloudwatchr](https://github.com/dewrich/cloudwatchr) | Spend alerting (this repo, downstream) | — |

CloudLedger lists CloudWatchr as a **downstream consumer** of its Costs API — the
reciprocal of this edge. See CloudLedger's
[context](https://github.com/dewrich/cloudledger/blob/main/docs/context/index.md#downstream-consumers)
for that side of the boundary.

> **POC scope.** Two containers, one database, one upstream dependency. Documentation-first:
> the design is reviewed here before any code is written — the same discipline CloudLedger follows.

<!-- archdocs:begin -->
## Relations

- **reads** → [cloudledger (CloudLedger Costs API)](../externals/cloudledger-costs-api.md) — Reads aggregated AWS spend from CloudLedger's Costs domain API to evaluate budget rules · cross-repo → https://github.com/dewrich/cloudledger
<!-- archdocs:end -->
