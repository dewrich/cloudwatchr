# 0001 — Record architecture decisions

## Context

CloudWatchr is designed documentation-first, as a downstream peer of CloudLedger. Decisions made
now — how we consume CloudLedger, where alert state lives, how we notify — will be hard to
reverse once code lands, and their *rationale* is what a future reviewer will lack.

## Decision

We record architecturally significant decisions as **MADR** records, trimmed to four
headings (Context, Decision, Consequences, Status), kept in
`docs/topic-guides/decisions/` in this repo. One decision per file, numbered. This mirrors
[CloudLedger's ADR practice](https://github.com/dewrich/cloudledger/blob/main/docs/topic-guides/decisions/0001-record-architecture-decisions.md)
so the two repos stay legible to the same reviewers.

## Consequences

- The "why" lives next to the code and diffs in PRs.
- A small tax on each significant change in exchange for durable rationale.

## Status

Accepted
