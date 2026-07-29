# ADR-0011 — Time, timezone and period policy

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human)

## Context

Decision D-08, raised by the VS-000 audit, blocked VS-007 (canonical
orders) and VS-014 (dashboard). Period assignment silently determines which
month a figure lands in — a report that says "March profit" is meaningless
until the timezone and the period boundary are fixed, and getting it wrong
moves money between periods without any error appearing anywhere.

The template already stores `created_at` timezone-aware, so the storage
half was settled by inheritance. The reporting half was not.

## Decision

### Storage

**All timestamps are stored timezone-aware in UTC.** This confirms existing
practice; it is not a change. `DateTime(timezone=True)` on every timestamp
column, no naive datetimes anywhere in the codebase.

### Reporting

**The reporting timezone is `Asia/Ho_Chi_Minh` (UTC+07:00, no daylight
saving).** Every business date — which period an order, refund, cost
effective date or settlement falls into — is determined by converting the
stored UTC instant into `Asia/Ho_Chi_Minh` and taking the local calendar
date.

The reporting timezone is stored as an organization-level setting,
defaulting to `Asia/Ho_Chi_Minh`, so that a future organization in another
region does not require a migration. It is **not** per-user: two people
looking at the same dashboard must see the same numbers.

### Periods

- **Weeks start on Monday** (ISO 8601).
- **The fiscal year matches the calendar year.** January to December, four
  calendar quarters.
- A **day** is `[00:00:00, 24:00:00)` local time — half-open, so an instant
  belongs to exactly one day.
- A **month** is the local calendar month; a **quarter** is three of them.
- All period ranges are half-open, consistent with the effective-dating
  convention in
  [`../HISTORICAL-DATA-POLICY.md`](../HISTORICAL-DATA-POLICY.md).

### Display

Any figure keyed on a date states **which** date drives it — order date,
refund date, settlement date, payout date. "March profit" without that
qualifier is not an acceptable label
([`../../product/UX-PRINCIPLES.md`](../../product/UX-PRINCIPLES.md) §9).

### Imports

A marketplace export carrying a naive local timestamp is interpreted in the
**marketplace's** timezone, converted to UTC on ingest, and the assumed
source timezone is recorded on the import profile. It is never assumed to
be UTC and never assumed to match the reporting timezone.

## Alternatives considered

- **Report in UTC** — rejected. A UTC day boundary falls at 07:00 local,
  so a seller's evening orders would land in the next day's figures. The
  numbers would be defensible and useless.
- **Per-user reporting timezone** — rejected. Two users comparing the same
  dashboard would see different totals for the same period, which destroys
  the shared-truth property the product exists to provide.
- **Sunday week start** — rejected, though some marketplace seller-centre
  reports use it. ISO 8601 Monday is the wider convention, and where a
  marketplace's own weekly report disagrees, that becomes a documented
  reconciliation difference rather than a hidden one.
- **Fiscal year offset from the calendar** — rejected; no business reason
  was identified, and it would complicate every period comparison.
- **Hard-coding the timezone rather than storing it** — rejected. Storing
  it costs nothing now and avoids a migration later.

## Consequences

- Positive: period assignment is deterministic and testable, and the tests
  can target exact boundary instants.
- Positive: `Asia/Ho_Chi_Minh` has no daylight saving, so there are no
  ambiguous or non-existent local times — a real source of defects in
  DST-observing zones, avoided by the choice rather than handled.
- Trade-off: every aggregation query must convert UTC to local before
  grouping. This must be done in one shared place, not repeated per query,
  or the two will eventually disagree.
- Trade-off: a future organization in a DST zone will need the ambiguous-
  time cases handled. Deferred deliberately; the column exists.
- Obligation: VS-007 and VS-014 must include boundary tests — an order at
  `16:59:59Z` and one at `17:00:00Z` fall on different local days and must
  land in different periods.
- Obligation: the KPI dictionary states the date basis for every
  period-keyed KPI.

## Related

[`../TARGET-ARCHITECTURE.md`](../TARGET-ARCHITECTURE.md) §6,
[`../DATA-MODEL.md`](../DATA-MODEL.md) §2,
[`../HISTORICAL-DATA-POLICY.md`](../HISTORICAL-DATA-POLICY.md),
[`../../ba/06-KPI-DICTIONARY.md`](../../ba/06-KPI-DICTIONARY.md),
[`../../delivery/slices/VS-007-ORDERS.md`](../../delivery/slices/VS-007-ORDERS.md),
[`../../delivery/slices/VS-014-PROFITABILITY-DASHBOARD.md`](../../delivery/slices/VS-014-PROFITABILITY-DASHBOARD.md),
decision D-08 in [`../../ba/08-BUSINESS-RULES.md`](../../ba/08-BUSINESS-RULES.md).
