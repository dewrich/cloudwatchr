---
id: ext/slack
kind: external
system: cloudwatchr
name: Slack
tech: Slack Incoming Webhooks
owner: external
tags: [notification, chatops]
relations: []
---

Slack workspace channels where CloudWatchr posts alerts. The
[Notifier](../containers/notifier.md) writes to a per-channel Incoming Webhook — one channel for
routine threshold/spike alerts, an escalation channel paired with PagerDuty for budget breaches.
Write-only from CloudWatchr's side.
