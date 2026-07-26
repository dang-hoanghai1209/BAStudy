# Historical Data Policy

**Status:** Decided (principles, effective-dating, append-only correction) /
Open (recalculation trigger, period locking, retention windows).

SellerMargin's core promise is that a number can be trusted and traced.
That promise is broken the moment a change made today silently alters what
the system reported yesterday. This document states how history is
protected, and it outranks convenience in every slice.

## 1. The four principles

1. **Never overwrite a value that has history.** Cost, fee rules and
   mappings are appended and superseded, not edited in place.
2. **Never silently rewrite a reported figure.** A recalculation that
   changes a previously reported number is an explicit, audited, attributed
   action — never a side effect of editing an input.
3. **Every calculated figure records its inputs *and their versions*.**
   Storing "COGS = 250,000" is insufficient; store which `cost_history`
   record, with which effective range, and which `fee_rule` version.
4. **A raw source row is immutable.** Corrections are made by re-import or
   by an audited business-layer change, never by editing the source layer.

## 2. Effective-dated (temporal) entities

These entities are time-versioned rather than mutable:

| Entity | Effective range | Introduced in |
|---|---|---|
| `cost_history` | `effective_from` / `effective_to` (null = current) | VS-006 |
| `fee_rule` | `effective_from` / `effective_to` (null = current) | VS-008 |
| `channel_sku_mapping` | effective-dated if remapping is retroactive (Open — D-06) | VS-003 |

Invariants, enforced in the database where possible and always in tests:

- Ranges for the same key must never overlap.
- At most one open-ended (`effective_to IS NULL`) record per key.
- A resolution query returns the record whose range **covers the business
  date of the transaction**, not the most recently created record.
- If no record covers the date, that is a reportable exception — never a
  silent zero and never a fallback to the nearest record.

Half-open ranges are used: `effective_from` inclusive, `effective_to`
exclusive. This makes adjacent ranges unambiguous at the boundary instant.

## 3. Correcting a past record

To correct an effective-dated record, do **not** update it. Instead:

1. Insert a superseding record covering the corrected range.
2. Close the previous record by setting its `effective_to`.
3. Record actor, timestamp and reason.
4. Emit an audit event ([`../security/AUDIT-POLICY.md`](../security/AUDIT-POLICY.md)).

The previous record remains readable. Any calculation that referenced it
continues to point at it, and continues to explain itself correctly, until
an explicit recalculation is run.

## 4. Recalculation

**Open decision (D-12) — blocks VS-009.** The following must be answered
before the profit engine is implemented:

- Does correcting a past cost trigger recalculation of historical orders
  automatically, on demand, or never? (Mandatory question #8 in
  [`../ba/08-BUSINESS-RULES.md`](../ba/08-BUSINESS-RULES.md))
- Who may trigger a recalculation that changes reported figures?
- Is a reason mandatory?

Whatever the answer, these constraints already hold:

- A recalculation **appends a new ledger entry**; it never mutates the
  prior one. The previous result stays readable and attributable.
- Every ledger entry names the calculation version that produced it.
- A recalculation that changes a reported figure is auditable, with actor,
  timestamp, scope and reason.
- Recalculating a locked period is refused unless the period is explicitly
  unlocked first, by someone permitted to do so.

## 5. Period locking

**Open decision (D — mandatory question #9) — blocks VS-012.** The intent:
once a period is reconciled and reported, it is closed to further change so
that reported figures stay stable.

Constraints already agreed:

- A lock is per organization, per shop, per period.
- Unlocking is a permissioned, audited action with a mandatory reason.
- Late-arriving transactions for a locked period (mandatory question #14)
  need a defined destination — either an adjustment in the open period or a
  controlled reopen. This must be decided, not discovered during coding.

## 6. Deletion

Business records are **not hard-deleted**. They are deactivated or
superseded so that historical references stay resolvable.

This is a deliberate divergence from the inherited template, which
hard-deletes with `ON DELETE CASCADE` (`item.owner_id` → `user.id`). That
pattern is acceptable for the template's demo entity and is **not** to be
copied onto any SellerMargin business table.

| Record type | On "delete" |
|---|---|
| Product / SKU / mapping | Deactivate; excluded from new selections, still resolvable on historical records |
| Cost / fee rule | Close the effective range; never remove |
| Import batch | Soft-delete with reason; source rows retained per the retention policy |
| Order / order line | Never deleted; status change only |
| Calculation ledger entry | Never deleted; superseded by a new entry |
| Audit event | Never deleted or modified, by any API |
| User | Deactivate (`is_active = false`) rather than delete, so audit attribution survives |

Hard deletion, when genuinely required (for example a privacy erasure
request), is an operational procedure with explicit human sign-off recorded
in the pull request — not an API capability.

## 7. Retention

**Open decision (D-13).** Retention windows for raw uploaded files, import
source rows and audit events are not yet set, and they interact with the
personal data that real marketplace exports contain. Until decided:

- Raw files and source rows are retained indefinitely (nothing is purged).
- No purge job is implemented.
- See [`../security/DATA-PRIVACY.md`](../security/DATA-PRIVACY.md) for what
  must be decided and why it cannot be deferred past VS-004.

## 8. Traceability requirement

Every number the system displays must be traceable to:

1. the source import row(s) it derives from,
2. the cost history record(s) used, with effective dates,
3. the fee rule version(s) applied,
4. the calculation version that produced it.

This is a hard acceptance criterion for VS-009 and VS-015, not an
optimization, and it is one of the project's stated depth criteria in
[`../ba/09-UAT-STRATEGY.md`](../ba/09-UAT-STRATEGY.md).

## 9. Tests that must exist

Any slice touching historical data must prove, not assert:

- adding a cost does not mutate an existing cost row;
- effective ranges for one key never overlap;
- resolution at a date returns the covering record, including at exact
  boundaries;
- a missing covering record produces a reportable exception, not a zero;
- a recalculation appends rather than overwrites;
- re-importing identical data does not duplicate business records.
