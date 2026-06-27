# 0007. Use PostgreSQL for the order store

**Status:** 🟢 Accepted — 2026-03-12

## Context and problem

The order service needs a primary datastore for orders, line items, and their
status transitions. Orders are read and written together as a unit, are queried
by customer and by status, and must never lose a committed write. We expect on
the order of 200 writes/second at peak in the first year. The team has prior
production experience with relational databases but none operating a wide-column
store. We need to commit to one datastore before building the persistence layer.

## Decision drivers

- Strong consistency and transactional writes across an order and its line items.
- Flexible querying by customer, status, and date range without pre-modelling
  every access pattern.
- Team operational familiarity and time to first production deploy.
- Cost and scaling headroom for the first two years (`<TODO: confirm 2-year
  volume projection with product>`).

## Considered options

### Option A — PostgreSQL (managed, e.g. RDS/Cloud SQL)

- **Pro:** ACID transactions cover the order + line-item write as one unit, and
  joins/ad-hoc queries satisfy the by-customer and by-status drivers directly.
- **Pro:** The team already operates Postgres in production — fastest time to
  ship and a known operational model.
- **Con:** Horizontal write scaling needs sharding or read replicas later; not
  free past a single primary's ceiling.

### Option B — DynamoDB (wide-column)

- **Pro:** Effectively unbounded write scaling with predictable latency.
- **Con:** Access patterns must be modelled up front; the by-status and
  date-range queries need secondary indexes and are awkward to evolve.
- **Con:** No team operational experience — a learning curve that delays the
  first deploy and raises the risk of modelling mistakes.

### Option C — MongoDB (document)

- **Pro:** The order aggregate maps naturally to a single document.
- **Con:** Multi-document transactions exist but are less battle-tested for our
  team than Postgres; weaker fit for the ad-hoc relational queries we need.

## Decision

We chose **PostgreSQL**. It satisfies the consistency driver outright (the
order + line-item write is one transaction) and the querying driver without
pre-modelling every access pattern, and it is the option the team can run safely
today — meeting the time-to-deploy driver. The projected first-year write volume
sits well within a single managed primary, so the scaling ceiling is not a
near-term constraint.

## Consequences

- **Good:** One transactional write per order; rich querying for ops and support
  with no extra modelling; fastest path to production on familiar operations.
- **Bad:** A future write-scaling ceiling we will eventually hit; crossing it
  means read replicas then sharding, which is non-trivial.
- **Neutral:** Add a load test before launch to confirm the 200 writes/second
  assumption, and set an alert on primary write saturation to trigger a revisit.

## Links

- Supersedes none.
- Related: `0004-adopt-managed-databases.md`.
