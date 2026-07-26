# VS-017 — Audit Trail

**Slice ID:** VS-017 · **Depends on:** VS-006, VS-008, VS-016 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Note that the **table and emit helper ship in VS-001**,
> not here — see §Scope boundary.

## Business objective

Make it possible to reconstruct who changed what, when, and why, for every
business-significant action.

## BA learning outcome

Distinguishing an **audit trail** from application logging: logs serve
operators and may be lossy; audit events are business records —
structured, complete, attributable and immutable.

## Scope boundary — read this first

The `audit_event` table, its shape, and the emit helper are delivered in
**VS-001**, and every slice from VS-001 onward emits its own events as part
of its Definition of Done
([`../../security/AUDIT-POLICY.md`](../../security/AUDIT-POLICY.md) §5).

**This slice delivers the query API, the viewer UI, and retention.** It is
deliberately not the slice where auditing starts — back-filling emission
into a dozen shipped slices is risk R-17, and the mitigation is to never
create the situation.

## Business rules required — **pending**

| Question | Status |
|---|---|
| Retention period for audit events | **Open (P-7)** |
| Who may read the audit log | Decided — organization Owner only |
| Is a reason mandatory? | Decided per action — see the audit policy action list |
| Immutability guarantee | Decided — no API may modify or delete an event |
| Personal data in `before`/`after` payloads | Decided — redacted |

## Entities involved

`audit_event` (created in VS-001): actor, organization, action code, entity
type and ID, before/after diff, reason, timestamp, request correlation ID,
source.

## Expected API capabilities

Query audit events with filters (actor, action, entity, date range).
Entity-scoped audit history for a drawer. Export the audit log (itself
audited). Retention handling.

## Expected frontend screens

Audit log viewer with filters (`/settings/audit`, Owner-only); a per-entity
**history drawer** surfaced on cost, fee rule, mapping and reconciliation
screens — the same component in four places.

## Acceptance criteria

1. Every action on the auditable list produces exactly one event — not
   zero, not duplicates — including when reached by a second code path.
2. Audit events cannot be edited or deleted through any API, by any role.
3. A business rule change is fully reconstructable: what it was, what it
   became, who, when, why.
4. Audit reads are permission-gated and tenant-scoped; a superuser without
   a membership sees nothing.
5. Reading and exporting the audit log are themselves audited.
6. No personal-data value appears in any event payload.

## Test layers

Unit (event construction, diff capture, redaction); integration (one event
per auditable action; the second-code-path case; immutability attempts
rejected; cross-organization read returns nothing; reason-mandatory actions
refused without a reason); contract; frontend (viewer with many events,
none, loading, error); E2E (change a cost → find the event in the log).

## Dependencies

VS-006 and VS-008 (things worth auditing exist), VS-016 (who may read it).

## Risks

| Risk | Mitigation |
|---|---|
| **R-17** — back-filling emission into shipped slices | Emit from VS-001 onward; this slice only adds query and viewer |
| Before/after capture on large payloads has a storage cost | Capture changed fields only, never whole rows |
| Audit write failure silently swallowed | The transaction fails if the event cannot be written |

## CV evidence

"A business rule change has an audit trail" — one of the six depth
criteria. Compliance-aware thinking without over-claiming regulatory scope.

## Explicit exclusions

No cryptographic tamper-evidence (hash chaining or signing). No SIEM
export. No real-time alerting on audit events. No user-behaviour analytics.
No regulatory-compliance certification claims.
