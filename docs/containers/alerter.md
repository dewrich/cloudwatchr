---
id: cloudwatchr/alerter
kind: container
name: Alerter
system: cloudwatchr
tech: Go 1.23 / ECS Scheduled Task (EventBridge cron, every 15 min)
owner: observability-eng
tags: [batch, finops, evaluation]
relations:
  - to: ext/cloudledger-costs-api
    kind: reads
    desc: Pulls spend rollups and daily cost rows to evaluate against budget rules (cross-repo)
    tech: HTTPS / REST — GET /costs/rollups/by-service, /costs/daily (costs:view)
  - to: ext/aws-cognito
    kind: uses
    desc: Obtains a bearer token with costs:view so CloudLedger's API accepts the call
    tech: Cognito client-credentials / OIDC
  - to: cloudwatchr/alert-state-db
    kind: writes
    desc: Reads active rules; writes fired/cleared alert state (idempotent per rule × window)
    tech: pgx
---

# Alerter — container (C4 L2)

A single Go binary (`cloudwatchr evaluate`) run on a schedule. Each tick it:

1. Mints a [Cognito](../externals/aws-cognito.md) token carrying the `costs:view` scope.
2. Calls the [CloudLedger Costs API](../externals/cloudledger-costs-api.md) —
   `GET /costs/rollups/by-service` for the current window, plus `GET /costs/daily` when a rule
   needs day-grain detail.
3. Loads active **rules** from [`alert-state-db`](../contracts/alert-state-db.md) and evaluates
   each against the fetched spend.
4. **Upserts alert state** — a rule that crosses its threshold fires (or stays fired); one that
   falls back under clears. State is keyed on `(rule_id, window)` so re-runs are idempotent.

CloudWatchr **never** reads CloudLedger's database. The Costs API is the only sanctioned path,
exactly as CloudLedger's own docs require — see its
[Inventory API container](https://github.com/dewrich/cloudledger/blob/main/docs/containers/inventory-api.md).

## Rule shapes (POC)

| Rule kind | Fires when | Source field |
|---|---|---|
| `service_threshold` | amortized spend for a service over the window exceeds `$X` | `/costs/rollups/by-service` → `amortized_usd` |
| `account_budget` | an account's month-to-date spend exceeds its budget | `/costs/daily` summed by `account_id` |
| `spike` | today's daily spend is `N×` the trailing 7-day median | `/costs/daily` |

## Responsibilities

- **Read-only upstream.** The Alerter only ever issues `GET`s against CloudLedger; it cannot
  mutate cost data (nor should it want to).
- **Freshness-aware.** CloudLedger responses carry `collected_at`; the Alerter records it on
  each fired alert so on-call can tell how stale the underlying sweep was.
- **Idempotent.** Evaluation is a pure function of (rules, fetched spend); re-running a tick
  produces the same alert state.

## Failure modes

- **CloudLedger API unreachable / 5xx** → the tick is skipped and retried next cron; no alert is
  cleared on missing data (fail closed — a silent evaluator must not look like "all healthy").
- **Cognito unreachable** → cannot mint a token → same skip-and-retry path.
- **Stale upstream sweep** (`collected_at` old) → the Alerter still evaluates but tags the alert
  `stale_source` so responders discount it.

<!-- archdocs:begin -->
## Relations

- **reads** → [ext/cloudledger-costs-api](../externals/cloudledger-costs-api.md) — Pulls spend rollups and daily cost rows to evaluate against budget rules (cross-repo)
- **uses** → [ext/aws-cognito](../externals/aws-cognito.md) — Obtains a bearer token with costs:view so CloudLedger's API accepts the call
- **writes** → [cloudwatchr/alert-state-db](../contracts/alert-state-db.md) — Reads active rules; writes fired/cleared alert state (idempotent per rule × window)
<!-- archdocs:end -->
