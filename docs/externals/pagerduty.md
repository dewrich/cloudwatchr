---
id: ext/pagerduty
kind: external
system: cloudwatchr
name: PagerDuty
tech: PagerDuty Events API v2
owner: external
tags: [notification, on-call, incident]
relations: []
---

PagerDuty service that pages on-call for high-severity spend events. The
[Notifier](../containers/notifier.md) triggers an incident when an `account_budget` rule breaches
and resolves it when the alert clears (Events API v2 `trigger`/`resolve`, de-duplicated on the
alert's `dedup_key`).
