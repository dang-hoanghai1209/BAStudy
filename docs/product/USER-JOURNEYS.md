# User Journeys

**Status:** Draft — target journeys, not implemented. Personas referenced
here are defined in [`PERSONAS.md`](PERSONAS.md); none are interview-validated.

Each journey names the slices that make it possible. A journey is not
"done" until every step works without a manual workaround.

---

## J1 — First-run setup (Owner, once)

**Trigger:** a new seller signs up.
**Slices:** VS-001, VS-002

```
Sign up / log in
  → create organization (name, currency — immutable after this)
    → create shops, one per marketplace presence
      → invite the Analyst, assign role
        → land on a dashboard that honestly says "no data yet"
```

**Succeeds when** the Owner reaches a working empty state and knows the
next action ("import your first file"), rather than a blank screen or
zeroes presented as facts.

**Failure modes to design against**
- Currency chosen carelessly and immutable afterwards → make the
  consequence explicit at the point of choice, not in a tooltip.
- Empty dashboard indistinguishable from a broken dashboard.

---

## J2 — Weekly data load (Analyst, the highest-frequency journey)

**Trigger:** marketplace exports are ready.
**Slices:** VS-003, VS-004, VS-005

```
Import Center → upload CSV
  → system detects encoding, delimiter, headers
    → column mapping (pre-filled from the saved profile for this marketplace)
      → preview first rows
        → confirm → background processing
          → batch summary: imported / skipped / failed, with reasons
            → fix causes (map SKUs, add missing costs)
              → reprocess
                → clean batch
```

**Succeeds when** a routine weekly load is a few clicks and no re-keying,
and every failed row explains itself in business terms.

**Failure modes to design against**
- Re-uploading the same file double-counts revenue. Idempotency is a
  business requirement here, not a technical nicety.
- An unmapped SKU blocks the whole file instead of parking those rows.
- The Analyst cannot tell whether a background import is running, finished,
  or died.
- An error message names a database constraint instead of the problem.

---

## J3 — Keeping costs current (Analyst, monthly or on price change)

**Trigger:** a supplier price changes, or the missing-cost report is
non-empty.
**Slices:** VS-002, VS-006

```
Missing-cost report → open SKU
  → view cost timeline (what it was, when)
    → add new cost with effective-from date
      → confirm: historical orders are unaffected
```

**Succeeds when** the Analyst can add a cost without fear of changing last
quarter's reported profit, and can *see* that it did not change.

**Failure modes to design against**
- The UI implies "cost" is a single editable field. It must read as a
  timeline from the first screen.
- Correcting a past cost silently rewrites reported history — the outcome
  must be stated before confirming, and it is a permissioned action.

---

## J4 — Understanding a number (Owner, whenever something looks wrong)

**Trigger:** a figure on the dashboard is surprising. This is the journey
that determines whether the product is trusted.
**Slices:** VS-009, VS-014, VS-015

```
Dashboard → profit for a shop looks wrong
  → drill into the period → SKU breakdown
    → open the worst SKU → its orders
      → open one order → profit breakdown
        → each term shows its source:
             COGS      → which cost record, effective from when
             Fees      → which fee rule, which version
             Revenue   → which source row, from which import batch
          → "this fee rule changed on the 14th" → the explanation
```

**Succeeds when** the Owner ends with an explanation rather than doubt, in
at most four steps.

**Failure modes to design against**
- Drill-down totals not matching the aggregate they came from. This is the
  classic reporting defect and needs an invariant test, not spot checks.
- A dead end where a number has no explanation.
- Personal data exposed in the raw-row view — masked by default
  ([`../security/DATA-PRIVACY.md`](../security/DATA-PRIVACY.md)).

---

## J5 — Month-end reconciliation (Analyst → Owner)

**Trigger:** the marketplace publishes a settlement statement.
**Slices:** VS-011, VS-012, VS-013

```
Import settlement file
  → run reconciliation for shop + period
    → matched / unmatched / variance
      → work the variance queue, classified by cause
        → manually match what the rules could not
          → Owner reviews and confirms
            → period locked → figures stable
```

**Succeeds when** every statement line ends in exactly one state, and the
Owner can confirm knowing nothing is silently unaccounted for.

**Failure modes to design against**
- Lines that quietly match nothing and disappear from view.
- Confirmation available to whoever happens to be logged in — confirming
  is Owner-only and audited.
- A late transaction arriving after lock, with no defined destination
  (mandatory question #14 — still open).

---

## J6 — Period close and handover (Owner → Accountant)

**Trigger:** the period is reconciled and locked.
**Slices:** VS-017, VS-018

```
Generate period report
  → export CSV
    → hand to the accountant
      → figures do not move afterwards
        → if they must, the audit trail explains why
```

**Succeeds when** the exported figures equal the on-screen figures exactly
and remain stable.

**Failure modes to design against**
- Export and screen computed by different code paths and disagreeing.
- Export containing personal data without permission or audit.

---

## J7 — Investigating a data-quality problem (Analyst, ongoing)

**Trigger:** the exception queue is non-empty.
**Slices:** VS-005, VS-006, VS-008, VS-013

```
Exception inbox (unmapped SKU, missing cost, unrecognised fee, variance)
  → filter by type → open one
    → jump to the offending record
      → fix the cause
        → exception closes; dashboard health tile improves
```

**Succeeds when** fixing the underlying cause closes the exception without
a separate bookkeeping step, and the health tile is a live measure rather
than decoration.

---

## J8 — Demo walkthrough (portfolio reviewer)

**Slices:** J1 → J2 → J3 → J4 → J5, on synthetic data.

The single end-to-end scenario the project must be able to demonstrate:
import a file, map a SKU, set a cost, calculate profit, trace one number to
its source, import a settlement, reconcile it, explain a variance.

This is the "demo of one real business scenario end to end" criterion in
[`../ba/09-UAT-STRATEGY.md`](../ba/09-UAT-STRATEGY.md). All demo data is
synthetic and labelled as such.

---

## Journey-to-slice coverage

| Journey | Complete after |
|---|---|
| J1 First-run setup | VS-002 |
| J2 Weekly data load | VS-005 |
| J3 Keeping costs current | VS-006 |
| J4 Understanding a number | VS-015 |
| J5 Month-end reconciliation | VS-013 |
| J6 Period close and handover | VS-018 |
| J7 Data-quality investigation | VS-013 |
| J8 Demo walkthrough | VS-015 (partial), VS-018 (full) |
