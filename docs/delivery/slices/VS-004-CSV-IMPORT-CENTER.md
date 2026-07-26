# VS-004 — CSV Import Center (ingest and staging)

**Slice ID:** VS-004 · **Depends on:** VS-001, VS-003 · **Status:** Not started · **Level:** Specified

## Business objective

Get marketplace order exports into the system reliably and repeatably,
without writing unvalidated data into business tables. This is the
project's own highest-priority capability.

## BA learning outcome

Separating **source, canonical and business layers**, and specifying
idempotency in business terms — "re-uploading the same file must not
double-count revenue" — rather than as a technical detail.

## Business rules required

| Rule | Status |
|---|---|
| What makes two uploads "the same": file hash, or natural key of order + order item | **Open (D-09)** — blocks coding |
| Can a partially-processed batch be retried? | **Open** |
| Maximum file size | **Open** |
| Supported encodings and delimiters; behaviour when detection is ambiguous | **Open** |
| Raw source rows are immutable | Decided |
| **Personal-data handling and retention** | **Open (D-13) — hard blocker**, see below |

### D-13 is a hard blocker for this slice

Real marketplace exports contain buyer names, addresses and phone numbers.
This slice decides whether that data is stored verbatim, masked, or dropped
at ingest — and for how long. Building on the current default (store
everything, forever, because nothing was decided) is the highest-risk
option available. See [`../../security/DATA-PRIVACY.md`](../../security/DATA-PRIVACY.md) §4.

## Entities involved

`import_batch` (file metadata, hash, status, counts, actor, shop),
`import_source_row` (raw row, immutable, row number, batch reference),
`import_profile` / `column_mapping` (saved per marketplace + file type),
`marketplace_file_type`.

`import_batch` and `import_source_row` carry both `organization_id` and
`shop_id`.

## Expected API capabilities

Upload file (multipart — `python-multipart` is already a dependency).
Detect encoding, delimiter and headers. Preview the first N rows. Save and
reuse a column-mapping profile. Start processing. Batch status. Batch list
and history. Download the original file (permissioned).

Processing runs via FastAPI `BackgroundTasks`, not Celery
([`../../architecture/TARGET-ARCHITECTURE.md`](../../architecture/TARGET-ARCHITECTURE.md) §8).
Because an in-process task does not survive a restart, batch status must
make an interrupted batch **visibly incomplete**, never silently lost.

## Expected frontend screens

| Screen | Route |
|---|---|
| Import Center / batch history | `/imports` |
| Import wizard: upload → detect → map → preview → confirm | `/imports/new` |
| Batch detail with counts and status | `/imports/$batchId` |
| Column-mapping profiles | `/imports/profiles` |

The wizard is the most complex UI in the early roadmap: multi-step,
long-running background work, and a reprocess loop. Budget for it.

## Acceptance criteria

1. Uploading the identical file twice yields one effective dataset and a
   clear "already imported" outcome — not a duplicate and not a silent
   no-op.
2. A saved column profile pre-fills the mapping on the next upload from
   the same marketplace and file type.
3. Raw rows are retrievable for any batch, with personal-data columns
   handled per the D-13 decision.
4. Processing does not block the request; the batch shows a live status
   and reaches a terminal state.
5. A batch interrupted by a restart is visibly incomplete and can be
   reprocessed.
6. Upload, processing and reprocessing emit audit events.
7. Import is shop-scoped: a batch belongs to one shop within one
   organization.

## Test layers

- **Unit** — dedupe key derivation; delimiter and encoding detection;
  column-profile application.
- **Integration** — upload; re-upload identical; malformed file; oversized
  file; unsupported encoding; role matrix; cross-organization 404.
- **Contract** — client regenerated.
- **Frontend** — wizard step states; in-progress, failed and partial batch
  states.
- **E2E** — upload a synthetic fixture CSV, see the batch appear with counts.

All fixtures are synthetic. **No real marketplace export is ever committed
to this repository**, including as a test fixture.

## Dependencies

VS-001 (org/shop scoping, audit), VS-003 (channel SKUs land through import).

## Risks

| Risk | Mitigation |
|---|---|
| **D-13 unresolved** — building a customer-data store by accident | Hard gate on Definition of Ready |
| Background processing loses a batch on restart | Explicit status; reprocess path; documented limitation |
| Large files exhaust the 8 GB local target | File-size limit; streaming parse rather than loading whole files |
| No real sample files (B-5) — built against invented shapes | Obtain at least one anonymised real export before starting |
| Slice sprawl: validation rules creep in from VS-005 | Exclusions below are binding |

## CV evidence

A four-layer data flow, implemented and documented. **Idempotent import
proven by test** — one of the project's stated depth criteria. A saved
column-profile design that shows real understanding of how operations
people actually work.

## Explicit exclusions

No marketplace API sync. No scheduled imports. No Excel/XLSX (CSV only).
No validation rules or error reporting (VS-005). No order creation
(VS-007). No settlement files (VS-011).
