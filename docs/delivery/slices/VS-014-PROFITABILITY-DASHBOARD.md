# VS-014 — Profitability Dashboard

**Slice ID:** VS-014 · **Depends on:** VS-009, VS-010, VS-013 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.

## Business objective

Show profit, margin and data health at a glance, per shop, channel, SKU and
period — replacing the placeholder dashboard inherited from the template.

## BA learning outcome

Translating a KPI dictionary into screens where **every displayed number
has exactly one documented definition**, and where incomplete data is
visible rather than hidden behind a confident figure.

## Business rules required — **pending**

| Question | Status |
|---|---|
| **D-08** — period definitions: calendar vs. fiscal, week start, reporting timezone | **Open — blocker** |
| Which order statuses are included in each KPI | **Open** |
| How missing/unknown data is represented: zero, excluded, or flagged | **Open** |
| Comparison-period semantics (vs. previous period, vs. same period last year) | **Open** |
| Every KPI on screen exists in the KPI dictionary with a matching formula | Decided — maintenance rule |

## Entities involved

**None new.** Read models and PostgreSQL views over the calculation ledger,
orders and refunds. Plain SQL and views first; materialized views only if a
measured latency problem justifies the staleness trade-off
([`../../architecture/TARGET-ARCHITECTURE.md`](../../architecture/TARGET-ARCHITECTURE.md) §9).

## Expected API capabilities

KPI summary for a period with filters. Time series of profit and margin.
Top and bottom SKUs by profit. Channel comparison. Data-health tile
(unmapped SKU rate, missing-cost count, open exceptions, unreconciled
amount).

## Expected frontend screens

Replace `frontend/src/routes/_layout/index.tsx` — currently a greeting
only — with KPI tiles, a trend chart, channel and SKU breakdown tables, a
filter bar, and explicit loading, empty and error states.

No chart library is chosen yet. Selecting one is a small ADR, and the
result must satisfy money formatting, colour-blind safety, and both light
and dark themes.

## Acceptance criteria

1. Every tile's figure matches its KPI dictionary definition exactly.
2. A period with no data shows an honest empty state — not zeros presented
   as facts.
3. Filters (shop, channel, date range) apply consistently across every
   widget.
4. Dashboard figures reconcile exactly with the VS-015 drill-down.
5. Incomplete data is visible: coverage is shown where a figure is based on
   a subset ("based on 812 of 1,004 orders").
6. Stale figures — where inputs changed since calculation — are marked.

## Test layers

Unit (aggregation SQL against fixture data; period boundaries; coverage
calculation); integration (endpoints with filters; tenant scoping; role
matrix); contract; frontend (all four states per widget — the state most
likely to be wrong here is empty vs. zero); E2E (load dashboard, change
filter, assert a known figure).

## Dependencies

VS-009 (profit), VS-010 (refunds) for correctness; VS-013 for the health
tile.

## Risks

| Risk | Mitigation |
|---|---|
| Shipping a dashboard before the formula is trustworthy produces confident wrong numbers — the exact failure the product exists to prevent | Hard dependency on VS-009 UAT passing |
| Query performance on realistic volumes untested locally | Test against a seeded dataset of realistic size before declaring done |
| Zero and "no data" rendered identically | Acceptance criterion 2; component tests for both |
| KPI formula drifts from the dictionary | Same-PR maintenance rule; a test asserting the documented formula |

## CV evidence

A KPI dictionary traced end to end from definition to pixel; a dashboard
that is honest about incomplete data, which is rarer than one that looks
polished.

## Explicit exclusions

No custom report builder (VS-018). No saved user dashboards. No scheduled
email digests. No AI narrative summary. No forecasting. No drill-down
(VS-015).
