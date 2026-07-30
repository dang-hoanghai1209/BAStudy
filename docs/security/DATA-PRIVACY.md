# Data Privacy Policy

**Status:** Decided (P-1, the core storage question — 2026-07-26, by the
human owner) / Open (P-4 – P-7 details, which follow from it and are
settled in their owning slices). **VS-004 is no longer blocked by this
document.**

## 0. The decision (P-1) — hash/truncate at ingest

**Decided 2026-07-26 by the human owner:** buyer personal data is
**hashed or truncated at ingest and never stored in full.**

Concretely:

- During column mapping (VS-004), columns carrying personal data (§6) are
  identified on the import profile.
- At ingest, each such value is reduced to: a **one-way hash** (for
  dedupe/idempotency purposes, since buyer identity can be part of a
  natural key) plus a **masked display value** (e.g. `Ng*** V** A***`,
  `09** *** 123`) for traceability display.
- The full original value is not written to `import_source_row`, and —
  because the value must exist nowhere — **the original uploaded file is
  not retained verbatim**: it is either scrubbed of personal-data columns
  before storage, or only the scrubbed representation is stored. "Download
  original file" (VS-004) returns the scrubbed form, and says so.
- Free-text note columns, which routinely smuggle addresses and phone
  numbers, default to being treated as personal data and are masked
  wholesale unless the import profile explicitly says otherwise.

Consequences accepted with this choice:

- A source row can never be re-read in full after ingest. If a column was
  mis-mapped, the remedy is **re-import from the seller's own copy**, not
  recovery from ours. This is the deliberate price of never becoming a
  customer database.
- P-2 and P-3 (retention windows for files and source rows) lose most of
  their urgency: what is retained no longer contains personal data in
  full. Retention windows are still to be set (storage hygiene, audit
  needs) but they no longer gate VS-004.
- P-4 and P-5 (who may view/export unmasked personal data) are largely
  mooted: **there is nothing unmasked to view or export.** The masked
  display value is visible wherever the source row is visible. The
  `export.personal_data` audit event is retained for the masked-data case
  and for any future policy change.
- P-6 (deletion requests): a request can be honoured by deleting the hash
  and masked value for the matching rows — feasible precisely because
  nothing else exists. Procedure still to be written at VS-019.

## 1. Why this exists

The pre-audit documentation set contained no privacy position at all, while
simultaneously proposing to retain raw marketplace exports indefinitely.
That combination is not tenable: **real marketplace order exports contain
personal data about the seller's customers** — typically buyer name,
delivery address, phone number, sometimes email, and occasionally partial
payment identifiers.

SellerMargin does not need any of it. Profit calculation needs SKUs,
quantities, amounts, fees and dates. But the CSV the seller uploads will
contain personal data anyway, and the import design proposed in VS-004
stores raw rows verbatim and keeps the original file.

That makes SellerMargin a processor of other people's customer data as a
side effect of its actual purpose. That must be a deliberate decision, not
an accident of the import design.

## 2. What the system is and is not

- SellerMargin is a **profitability and reconciliation** tool. Buyer
  identity has **no role in any calculation**.
- The seller is the data controller for their customers' data. SellerMargin
  processes it on their behalf.
- The project is a portfolio project, not a certified commercial service.
  It makes **no compliance claims** (GDPR, PDPA or otherwise). It should
  nonetheless behave sensibly, because the failure mode — leaking a real
  seller's customer list from a demo — is real regardless of certification.

## 3. Constraints that already hold

These apply from VS-004 onward regardless of the open decisions below.

1. **Do not promote personal data into business tables.** Order and
   order-line records store what the calculation needs. Buyer name, address
   and phone are not copied out of the source layer into `order`.
2. **Do not display personal data by default.** The drill-down in VS-015
   shows the source row for traceability; personal-data columns are masked
   unless explicitly revealed, and revealing is permissioned.
3. **Do not export personal data casually.** Any export containing
   personal data is permissioned and emits an
   `export.personal_data` audit event with a mandatory reason
   ([`AUDIT-POLICY.md`](AUDIT-POLICY.md)).
4. **Do not put personal data in logs, error messages, or Sentry.** A
   validation error names the row, the column and the rule — not the value,
   when the column is a personal-data column.
5. **Do not put personal data in audit `before`/`after` payloads.**
   Redact to a marker.
6. **Test fixtures use synthetic data only.** No real export is ever
   committed to the repository, including as a test fixture. This applies
   to the sample files needed for VS-004, VS-008 and VS-011: they must be
   anonymised before they enter the repository, and anonymisation happens
   outside it.
7. **`.env` and secrets are never reproduced in documentation.** The
   VS-000 audit reported only variable names and placeholder status.

## 4. Decision status

| ID | Decision | Status |
|---|---|---|
| P-1 | Personal-data columns at ingest | **Decided 2026-07-26 (human owner): hash/truncate, never stored in full** — §0 |
| P-2 | Retention of the original uploaded file | Open, no longer blocking — the stored form is scrubbed (§0). Set at VS-019 |
| P-3 | Retention of raw source rows | Open, no longer blocking — rows contain no full PII. Set at VS-019 |
| P-4 | Who may view unmasked personal data | Mooted by P-1 — nothing unmasked exists. Masked values follow normal source-row visibility |
| P-5 | Who may export personal data | Mooted by P-1 for unmasked data. Masked-data exports still emit `export.personal_data` |
| P-6 | Deletion request handling | Open — procedure written at VS-019; made feasible by P-1 |
| P-7 | Audit-event retention | Open — set at VS-017 |

## 5. Standing obligations

- VS-004 implements P-1 at ingest — personal-data identification is part
  of column mapping, and the hashing/masking behaviour has acceptance
  criteria and tests.
- No purge job exists yet; retention windows are set at VS-019.
- Any demo or portfolio screenshot uses synthetic data.

## 6. Personal-data column identification

Marketplace exports vary, but these column families are treated as personal
data wherever they appear, under any of their usual names:

buyer / customer name (incl. "recipient", "consignee"), delivery address
and its parts, phone number, email address, buyer account or username,
national ID, and any free-text note field (which routinely contains an
address or phone number even when the schema does not say so).

Free-text note fields are the most commonly missed case and should default
to being treated as personal data.

## 7. Out of scope

Compliance certification, data-processing agreements, consent management,
cross-border transfer analysis, and pseudonymisation key management. If
SellerMargin ever becomes a real service, each becomes a real requirement.
