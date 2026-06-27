# 0012. Adopt asynchronous fulfilment via an outbox

**Status:** 🟡 Proposed — 2026-05-02

## Context and problem

When an order is paid, the order service calls the warehouse system inline to
create a fulfilment request. That synchronous call couples checkout latency and
availability to the warehouse: when the warehouse is slow or down, paid orders
fail at the last step and customers see errors despite a successful payment. We
want paid orders to be durable and fulfilment to proceed independently of
checkout. This RFC proposes the integration pattern; feedback is requested before
we commit.

## Decision drivers

- A paid order must never be lost if the warehouse is unavailable.
- Checkout latency should not depend on warehouse response time.
- Exactly-once-ish delivery to the warehouse (no duplicate fulfilments).
- Operational simplicity — avoid introducing infrastructure the team can't run.

## Considered options

### Option A — Transactional outbox + relay worker

- **Pro:** The order status change and the outbox row commit in one local
  Postgres transaction, so a paid order is never lost (satisfies durability).
- **Pro:** Checkout returns as soon as the row is committed; a background relay
  delivers to the warehouse, decoupling latency.
- **Con:** Requires a relay worker and idempotent delivery (dedup key) — more
  moving parts than a direct call.

### Option B — Direct call wrapped in retries

- **Pro:** Simplest change; no new components.
- **Con:** A long warehouse outage still loses or blocks orders once retries
  exhaust; checkout latency remains coupled. Fails the durability driver.

### Option C — Dedicated message broker (e.g. Kafka/SQS) at checkout

- **Pro:** Strong decoupling and replay.
- **Con:** Introduces broker infrastructure the team does not currently operate;
  heavier than the problem warrants today (operational-simplicity driver).

## Decision

We recommend **Option A — transactional outbox + relay worker**. It is the only
option that satisfies the durability driver (the status change and the intent to
fulfil commit atomically) while decoupling checkout latency, and it reuses the
Postgres we already run rather than adding broker infrastructure. We would adopt
Option C only if a second async integration appears and justifies a shared
broker. Feedback that would change this: evidence the relay's at-least-once
delivery can't be made idempotent on the warehouse side.

## Consequences

- **Good:** Paid orders survive warehouse outages; checkout latency is
  independent of the warehouse; no new infrastructure beyond a worker.
- **Bad:** We own a relay worker and must guarantee idempotent delivery via a
  dedup key the warehouse honours (`<TODO: confirm warehouse supports an
  idempotency key>`).
- **Neutral:** Add monitoring on outbox lag and a dead-letter path for messages
  the warehouse rejects repeatedly.

## Links

- Related: `0007-use-postgres-over-dynamodb.md` (the outbox lives in this DB).
- Supersedes: none — replaces the inline call introduced in `0009`.
