# CloudWatchr

CloudWatchr is a **cloud-spend alerting system**: it watches the cost data that
[CloudLedger](https://github.com/dewrich/cloudledger) already aggregates, evaluates it against
budget and threshold rules, and **raises an alert when expenses climb too high**. It owns no
cost data of its own — CloudLedger is the system-of-record for *what things cost*; CloudWatchr
is the system-of-record for *rules, thresholds, and which alerts have fired*. It is run by the
observability team as a scheduled evaluator plus an always-on notifier.

CloudWatchr is the **downstream peer of CloudLedger**: it reads the CloudLedger
[Costs domain API](https://github.com/dewrich/cloudledger/blob/main/docs/containers/inventory-api.md)
(`/costs/rollups/by-service`, `/costs/daily`) and never touches CloudLedger's database directly —
the API is the only sanctioned read path. See
[Related systems](context/index.md#related-systems) for the boundary between the two repos.

Architecture (C4) is the primary axis: [context/](context/index.md) is the system,
[containers/](containers/) the deployable units (the [Alerter](containers/alerter.md) evaluator
and the [Notifier](containers/notifier.md)), and the cross-repo dependency on CloudLedger is
modelled as the [CloudLedger Costs API](externals/cloudledger-costs-api.md) external. New here?
Read [How Our Documentation is Organized](reference/how-our-documentation-is-organized/index.md)
for the C4 × Divio scheme these docs follow — the same scheme CloudLedger uses.
