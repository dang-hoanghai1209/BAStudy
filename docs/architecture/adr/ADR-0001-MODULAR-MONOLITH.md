# ADR-0001 — Modular Monolith, Not Microservices

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human), carried over from prior research

## Context

SellerMargin is developed by a single person plus an AI coding agent,
targeting a machine with 8GB RAM, with a secondary goal of serving as a
Business Analyst portfolio piece. The project needs enough architectural
depth to be credible, without operational complexity that would slow
delivery or exceed what one developer can maintain.

## Decision

Build SellerMargin as a **modular monolith**: one deployable backend
application, internally organized into clear module boundaries
(identity, organizations, catalog, imports, orders, costs, fees,
profitability, finance, reconciliation, reports, audit — see
`docs/architecture/ARCHITECTURE.md`). Do not adopt microservices, a
message broker (e.g. Kafka), or container orchestration (e.g.
Kubernetes) for this project's current phase.

## Alternatives considered

- **Microservices per module** — rejected: operational overhead
  (service discovery, distributed tracing, inter-service contracts)
  provides no proportional benefit at this project's scale and would
  consume time better spent on business logic and BA depth.
- **Single unstructured monolith (no module boundaries)** — rejected:
  would erode maintainability and make it harder to reason about
  ownership and testing boundaries as the codebase grows.

## Consequences

- Positive: simpler local development (fits the "Lite runtime" in
  `ARCHITECTURE.md`), lower operational burden, faster iteration.
- Positive: module boundaries still give a credible answer to "how is
  this organized" without paying for distributed-systems complexity.
- Trade-off: if the project ever needs true independent scaling of one
  module, that would require a future migration — deliberately deferred,
  not a current concern.

## Related

[`../TARGET-ARCHITECTURE.md`](../TARGET-ARCHITECTURE.md) §2 (the target
module layout), [`../CURRENT-STATE.md`](../CURRENT-STATE.md) §3 (the
backend is currently **flat** — `app/modules/` does not exist), and
[`ADR-0009-BACKEND-MODULE-STRUCTURE.md`](ADR-0009-BACKEND-MODULE-STRUCTURE.md),
which schedules the move from flat to domain packages as a standalone
refactor rather than leaving it implied by this decision.

[`../../ba/01-PRODUCT-VISION.md`](../../ba/01-PRODUCT-VISION.md) — "What
SellerMargin is not".
