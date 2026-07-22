# How Our Documentation is Organized

## Overview

These docs follow the **same scheme as [CloudLedger](https://github.com/dewrich/cloudledger)**,
its upstream peer, so anyone who knows one repo can navigate the other. Two orthogonal axes:
**C4** answers *which thing is this about* (system → container → component); the
**[Divio Documentation System](https://documentation.divio.com)** answers *what kind of writing
is this*. Every page has both coordinates.

## The Quadrants

### Tutorials — learning-oriented

Take a newcomer by the hand and turn a learner into a user. Start here if you're new.

### Topic Guides / Explanation — understanding-oriented

Discussion of key concepts at a high level, with the background and the *why*. Start
here for concepts, decisions (ADRs), and discussions.

### Reference — information-oriented

Dry, factual technical reference: APIs, schemas, settings. Describes how the
component works and how to use it, assuming you know the key concepts.

### How-to Guides — problem-oriented

Recipes that walk you through addressing a specific problem or use-case.

## Layout — C4 is the primary axis

The top-level folders name the **C4 levels**, so the folder tree itself shows the
architecture:

- `context/index.md` — the **system** node (C4 L1).
- `containers/<name>.md` — the **container** nodes (C4 L2), one file each.
- `externals/<name>.md` — systems/services outside this repo's boundary. **A peer repo we
  depend on is modelled here** — see
  [`cloudledger-costs-api.md`](../../externals/cloudledger-costs-api.md), which represents
  CloudLedger's Costs API as an external of CloudWatchr.

## Cross-repo links

CloudWatchr and CloudLedger are **separate GitHub repositories**. Links that stay inside this
repo are relative (`../containers/alerter.md`); links that cross into CloudLedger use **absolute
`https://github.com/dewrich/cloudledger/...` URLs**, because relative links cannot hop between
repositories on GitHub. Every cross-repo edge is also recorded in a node's `relations:`
frontmatter with a `repo:` field so the boundary is machine-readable, not just prose.

See the service overview: [index.md](../../index.md).
