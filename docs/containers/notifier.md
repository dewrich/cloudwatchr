---
id: cloudwatchr/notifier
kind: container
name: Notifier
system: cloudwatchr
tech: Go 1.23 / Fiber (webhook receiver) + worker (delivery)
owner: observability-eng
tags: [http, delivery, alerting]
relations:
  - to: cloudwatchr/alert-state-db
    kind: reads
    desc: Reads newly fired/cleared alerts to deliver; marks them delivered
    tech: pgx (LISTEN/NOTIFY on state change)
  - to: ext/slack
    kind: uses
    desc: Posts alert messages to the configured Slack channels
    tech: Slack Incoming Webhooks
  - to: ext/pagerduty
    kind: uses
    desc: Opens/resolves incidents for high-severity budget breaches
    tech: PagerDuty Events API v2
---

# Notifier — container (C4 L2)

An always-on Go service that turns rows in [`alert-state-db`](../contracts/alert-state-db.md)
into deliveries. It watches for state transitions (fired, cleared) and dispatches them to the
right channel with de-duplication and escalation.

## Responsibilities

- **Route by severity.** `service_threshold`/`spike` → [Slack](../externals/slack.md);
  `account_budget` breaches → [PagerDuty](../externals/pagerduty.md) incident + Slack.
- **De-duplicate.** One delivery per alert transition; a rule that stays fired across ticks is
  not re-notified until it clears and re-fires (state lives in `alert-state-db`).
- **Resolve.** When a fired alert clears, the Notifier resolves the PagerDuty incident and posts
  an "all clear" to Slack.

## Failure modes

- **Channel down (Slack/PagerDuty 5xx)** → the delivery is retried with backoff; the alert row
  stays `undelivered` so nothing is silently dropped.

<!-- archdocs:begin -->
## Relations

- **reads** → [cloudwatchr/alert-state-db](../contracts/alert-state-db.md) — Reads newly fired/cleared alerts to deliver; marks them delivered
- **uses** → [ext/slack](../externals/slack.md) — Posts alert messages to the configured Slack channels
- **uses** → [ext/pagerduty](../externals/pagerduty.md) — Opens/resolves incidents for high-severity budget breaches
<!-- archdocs:end -->
