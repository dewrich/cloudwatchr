---
id: cloudwatchr/alert-state-db
kind: contract
subkind: db
system: cloudwatchr
name: Alert State DB
tech: PostgreSQL 16 / Amazon RDS
owner: observability-eng
tags: [postgres, system-of-record]
relations: []          # contract nodes are passive join points; no outbound edges
---

# Alert State DB — contract (Postgres system-of-record)

CloudWatchr's own system-of-record — for **rules** and **alert state**, *not* for cost data
(that belongs to [CloudLedger](https://github.com/dewrich/cloudledger)). The
[Alerter](../containers/alerter.md) is the sole writer; the [Notifier](../containers/notifier.md)
reads it to dispatch deliveries.

## Shape

| Table | Grain | Written by |
|---|---|---|
| `rule` | one row per alert rule (kind, target, threshold, window) | operators (via config/API) |
| `alert_state` | one row per `(rule_id, window)`; fired/cleared + `collected_at` of source | Alerter |
| `delivery` | one row per notification attempt (channel, status) | Notifier |

**Boundary note.** This store holds *no* AWS cost figures — only thresholds and the
fired/cleared status computed from spend fetched live from CloudLedger's Costs API. CloudWatchr
deliberately does not cache CloudLedger's data as a source of truth; it evaluates against fresh
reads each tick.
