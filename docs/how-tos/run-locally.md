# Run CloudWatchr Locally

CloudWatchr needs a reachable CloudLedger Costs API to evaluate against. Point it at either a
local CloudLedger or a shared dev instance.

```bash
# Where CloudLedger's Costs API lives (see its run-locally guide):
export CLOUDLEDGER_COSTS_URL=http://localhost:9081
export COGNITO_TOKEN=...   # a token with the costs:view scope
```

1. **Start CloudLedger first.** Follow
   [CloudLedger → Run Locally](https://github.com/dewrich/cloudledger/blob/main/docs/how-tos/run-locally.md)
   and confirm `GET $CLOUDLEDGER_COSTS_URL/costs/rollups/by-service` returns rows.
2. **Bring up CloudWatchr's `alert-state-db`** (Postgres) and load rules.
3. **Run one evaluation tick:**

   ```bash
   cloudwatchr evaluate --once
   ```

   The Alerter mints/uses `$COGNITO_TOKEN`, pulls rollups from CloudLedger, evaluates rules, and
   writes fired/cleared state.

4. **Run the Notifier** to see deliveries (dry-run prints instead of posting):

   ```bash
   cloudwatchr notify --dry-run
   ```

If step 1's curl fails, nothing downstream will work — CloudWatchr has no cost data of its own.
See [why we consume via the API](../topic-guides/decisions/0002-consume-cloudledger-via-api-not-db.md).
