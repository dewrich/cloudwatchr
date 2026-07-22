---
id: ext/aws-cognito
kind: external
system: cloudwatchr
name: AWS Cognito
tech: Amazon Cognito user pool (OIDC / client-credentials)
owner: external
tags: [aws, identity, oidc]
relations: []
---

The AWS Cognito user pool that fronts CloudLedger. CloudWatchr's
[Alerter](../containers/alerter.md) obtains a bearer token carrying the **`costs:view`** scope
here, then presents it to the
[CloudLedger Costs API](./cloudledger-costs-api.md). This is the *same* Cognito pool CloudLedger
validates against — CloudWatchr is issued a service principal with read-only cost scope, nothing
more.
