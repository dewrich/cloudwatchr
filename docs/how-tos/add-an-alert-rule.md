# Add an Alert Rule

A rule couples a CloudLedger cost dimension to a threshold and a notification route.

1. **Pick the rule kind** (see the [Alerter](../containers/alerter.md) table):
   `service_threshold`, `account_budget`, or `spike`.
2. **Identify the source field** in CloudLedger's Costs API. For a per-service cap, that's the
   `amortized_usd` returned by `GET /costs/rollups/by-service` — shapes are defined in
   CloudLedger's [OpenAPI spec](https://github.com/dewrich/cloudledger/blob/main/api/openapi/costs/v1/api-spec.yml).
3. **Insert the rule** into [`alert-state-db`](../contracts/alert-state-db.md):

   ```sql
   INSERT INTO rule (kind, target, threshold_usd, window, route)
   VALUES ('service_threshold', 'Amazon EC2', 5000, 'month', 'slack:#finops-alerts');
   ```

4. **Verify** on the next tick (or `cloudwatchr evaluate --once`): if EC2's month-to-date
   amortized spend from CloudLedger exceeds `$5000`, the rule fires and the
   [Notifier](../containers/notifier.md) posts to `#finops-alerts`.

Rules never contain cost data — only thresholds. The actual spend is read live from CloudLedger
each tick.
