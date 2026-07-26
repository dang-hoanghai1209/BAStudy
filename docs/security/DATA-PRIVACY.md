# Data Privacy Policy

**Status:** Open — this document states the problem, the constraints that
already hold, and the decisions required. **It is not yet a finished
policy, and the missing decisions block VS-004.**

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

## 4. Decisions required (blocking VS-004)

| ID | Decision | Options | Blocks |
|---|---|---|---|
| P-1 | Are personal-data columns **dropped at ingest**, **stored but masked**, or **stored in full**? | Drop is safest and loses re-processing ability; mask-on-read is a middle ground; store-in-full maximises traceability and risk | VS-004 |
| P-2 | How long is the **original uploaded file** retained? | e.g. 30 / 90 / 365 days / indefinitely | VS-004 |
| P-3 | How long are **raw source rows** retained after successful processing? | Same options; may differ from P-2 | VS-004 |
| P-4 | Who may **view** unmasked personal data in drill-down? | Owner only / Owner + Analyst / nobody | VS-015 |
| P-5 | Who may **export** rows containing personal data? | Owner only / Owner + Analyst / nobody | VS-018 |
| P-6 | How is a **deletion request** for one buyer handled? | Manual operational procedure vs. a built capability | VS-019 |
| P-7 | Retention period for **audit events** containing redacted references | | VS-017 |

**Recommendation for P-1 (owner to confirm):** identify personal-data
columns during column mapping in VS-004 and store them **hashed or
truncated** rather than verbatim, keeping a masked display value for
traceability. This preserves "trace a number back to its source row"
without the system becoming a customer database. Full-fidelity retention
should be an explicit opt-in per organization, not the default.

## 5. Until the decisions are made

- VS-004 must **not** be implemented on the assumption that raw rows are
  stored verbatim and forever. That is the current default only because
  nothing has been decided, and it is the option with the most risk.
- No purge job exists, so nothing expires today. That is a known gap, not
  a design choice.
- Any demo or portfolio screenshot must use synthetic data.

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
