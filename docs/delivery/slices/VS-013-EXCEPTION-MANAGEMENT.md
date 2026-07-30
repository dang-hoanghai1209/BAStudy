# VS-013 — Exception Management

**Slice ID:** VS-013 · **Depends on:** VS-005, VS-006, VS-008, VS-012 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.
> **Note the shared-shape obligation in §Risks** — it affects VS-005.

## Business objective

Give every unresolved problem — validation error, unmapped SKU, missing
cost, unrecognised fee, reconciliation variance — one queue with ownership
and a resolution state, so that detection turns into resolution.

## BA learning outcome

Designing an **operational workflow** (states, ownership, ageing) on top of
detection that already exists. Detection without a workflow produces noise;
this slice is the difference.

## Business rules required — **pending**

| Question | Status |
|---|---|
| Exception taxonomy and severity | **Open** |
| Lifecycle states and legal transitions | **Open** |
| Assignment rules — self-assign, or assigned by Owner? | **Open** |
| Is a resolution reason mandatory? | **Open** |
| Does fixing the underlying data auto-close the exception? | **Open** |
| Ageing and escalation thresholds | **Open** |

## Entities involved

`exception` (polymorphic reference to the source record, type, severity,
state, ageing), `exception_assignment`, `exception_comment`,
`exception_resolution`.

## Expected API capabilities

List exceptions with filters and counts by type, severity and age. Assign.
Comment. Resolve with a reason. Bulk-resolve. Exception metrics for the
dashboard health tile.

## Expected frontend screens

Exception inbox with saved filters and ageing indicators (`/exceptions`);
exception detail with drill-through to the offending record
(`/exceptions/$exceptionId`); bulk actions.

## Acceptance criteria

1. Every detector from VS-005, VS-006, VS-008 and VS-012 raises into the
   **same queue with a consistent shape**.
2. Resolving the underlying data auto-closes the exception, or explicitly
   requires manual close, per the recorded rule.
3. Assignment and resolution are attributable to a user with a timestamp.
4. Dashboard health-tile counts match the queue exactly.
5. Bulk-resolving N exceptions produces N audit events.

## Test layers

Unit (state machine transitions, including illegal ones; auto-close
conditions; ageing calculation); integration (raise from each of the four
sources; assign; resolve; bulk; role matrix; cross-org 404); contract;
frontend (inbox empty state — an empty queue is a *good* state and must
look like one — loading, error, large lists); E2E (fix an unmapped SKU →
its exception closes).

## Dependencies

Functionally VS-005, VS-006, VS-008 and VS-012. It **can** ship earlier
covering fewer sources — see the risk below.

## Risks

| Risk | Mitigation |
|---|---|
| **R-18** — retro-fitting a common exception shape after four detectors already emit their own formats | **Agree the shared shape in VS-005** and have each later detector emit it. Consider shipping a minimal exception queue with VS-005 and extending it here |
| Queue becomes noise if everything raises an exception | Severity and taxonomy decided first; not every warning is an exception |
| Auto-close hides a problem that was not really fixed | Auto-close re-checks the underlying condition rather than trusting the edit |

## CV evidence

Operational workflow design with an explicit state machine; data quality
tied to an owned, measurable process rather than a report nobody reads.

## Explicit exclusions

No email or Slack notifications. No SLA automation. No external ticketing
integration. No on-call routing. No new detectors — this slice manages
exceptions raised elsewhere.
