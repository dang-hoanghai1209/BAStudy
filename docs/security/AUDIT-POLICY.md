# Audit Policy

**Status:** Decided (principles, auditable action list) / Open (retention period).

An audit trail answers a business question — *who changed this number, when,
and why* — and is not the same thing as application logging. Logs are for
operators and are allowed to be lossy, unstructured and short-lived. Audit
events are business records: structured, complete, attributable and
immutable.

## 1. Principles

1. **Attributable.** Every event names the acting user, the organization,
   and the time. There is no "system" actor for a user-initiated action.
2. **Immutable.** No API may update or delete an audit event. Not for an
   Owner, not for a superuser.
3. **Complete for its list.** Every action on the auditable list below
   produces exactly one event — never zero, never duplicates.
4. **Reconstructable.** For a change, the event records enough to
   reconstruct what the value was and what it became.
5. **Emitted where the change happens.** Events are emitted in the domain
   service that performs the change, not in the route handler, so an action
   reachable by more than one path cannot be audited on only one of them.
6. **Never silently dropped.** If writing the audit event fails, the
   transaction fails. An audited action that succeeded without its event is
   worse than a failed action.

## 2. Event shape

| Field | Notes |
|---|---|
| `id` | UUID |
| `organization_id` | tenancy scope; null only for platform-level events |
| `actor_user_id` | the acting user; never null for user-initiated actions |
| `action` | stable machine code, e.g. `cost.superseded`, `reconciliation.confirmed` |
| `entity_type`, `entity_id` | what was affected |
| `before`, `after` | changed fields only, not whole rows; personal data redacted per [`DATA-PRIVACY.md`](DATA-PRIVACY.md) |
| `reason` | free text; **mandatory** for the actions marked below |
| `occurred_at` | timezone-aware UTC |
| `request_id` | correlation ID, ties the event to request logs |
| `source` | `ui`, `api`, or `system` (for scheduled/background work) |

`action` codes are stable identifiers. Renaming one breaks historical
queries, so treat a rename as a breaking change.

## 3. Auditable actions

Reason mandatory where marked **R**.

### Organization and access (VS-001, VS-016)

- `organization.created`, `organization.updated`
- `shop.created`, `shop.updated`, `shop.deactivated`
- `membership.invited`, `membership.role_changed` **R**, `membership.removed` **R**
- `user.superuser_granted` **R**, `user.superuser_revoked` **R**
- `role.permission_changed` **R**

### Catalog and mapping (VS-002, VS-003)

- `product.created`, `product.updated`, `sku.deactivated`
- `mapping.confirmed`, `mapping.rejected`
- `mapping.remapped` **R** — changes how historical orders aggregate

### Import (VS-004, VS-005)

- `import.uploaded`, `import.processed`, `import.reprocessed`
- `import.batch_deleted` **R**
- `import.profile_saved`, `import.profile_updated`

### Costs and fees (VS-006, VS-008)

- `cost.added`
- `cost.superseded` **R** — a correction to history
- `fee_rule.created`, `fee_rule.superseded` **R**, `fee_rule.deactivated` **R**

### Profit and reconciliation (VS-009 – VS-012)

- `calculation.run`
- `calculation.recalculated` **R** — when it changes a previously reported figure
- `settlement.imported`
- `reconciliation.run`, `reconciliation.manually_matched`, `reconciliation.manually_unmatched`
- `reconciliation.confirmed` **R**, `reconciliation.reopened` **R**
- `period.locked` **R**, `period.unlocked` **R**

### Exceptions and reporting (VS-013, VS-018)

- `exception.assigned`, `exception.resolved` **R**, `exception.reopened` **R**
- `report.exported` — including scope and row count
- `export.personal_data` **R** — any export containing buyer personal data

### Deliberately **not** audited

Reads of ordinary business data, login and logout (application logs and
Sentry cover these), and navigation. Auditing reads would swamp the trail
and hide the changes that matter. Reads of the audit log itself, and
exports of personal data, are the exceptions and are audited.

## 4. Access to the audit log

- Readable by organization **Owner** only, scoped to that organization.
- Platform superusers do **not** read organization audit logs without a
  membership — the two-axis rule in
  [`AUTHORIZATION-POLICY.md`](AUTHORIZATION-POLICY.md) applies here too.
- Reading and exporting the audit log are themselves audited.
- Filterable by actor, action, entity, and date range.

## 5. Implementation obligation, and the retro-fit risk

VS-017 delivers the audit log **viewer, query API and retention handling** —
but slices VS-003 onward already perform auditable actions.

**Emit events from the slice that introduces the action, not later.**
Adding an `audit_event` table and an emit helper is a small piece of VS-001;
back-filling emission into a dozen shipped slices is not. This is tracked
as risk R-17.

Consequently, VS-001 delivers:

- the `audit_event` table and migration,
- an emit helper used by domain services,
- events for the VS-001 action list,

and every later slice adds its own events as part of its Definition of Done.

## 6. Retention

**Open decision.** Audit events are retained indefinitely until a retention
period is set. This interacts with personal data that may appear in
`before`/`after` payloads — see [`DATA-PRIVACY.md`](DATA-PRIVACY.md).

Constraint that already holds: retention expiry, when defined, deletes
whole events on a schedule. It is never a mechanism for removing individual
inconvenient events.

## 7. Tests

Every slice adding an auditable action must prove:

- the action produces exactly one event with the correct code and actor;
- an action performed through a second code path also emits the event;
- attempting to modify or delete an event through any API fails;
- an audit read from another organization returns nothing;
- a reason-mandatory action is refused when the reason is missing.

## 8. Out of scope

Cryptographic tamper-evidence (hash chaining, signing), SIEM export,
real-time alerting on audit events, and user-behaviour analytics. Each
would be a new ADR.
