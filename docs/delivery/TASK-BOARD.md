# Task Board

**Status:** Live — current state of every slice. For sequencing rationale
see [`VERTICAL-SLICE-ROADMAP.md`](VERTICAL-SLICE-ROADMAP.md); for the specs
themselves see [`slices/`](slices/).

## In review

### VS-000 — Repository audit, baseline checks, documentation adoption

**Status:** Complete, awaiting human review of this commit
**Branch:** `docs/VS-000-master-plan`

Delivered: full audit of the inherited repository; the documentation tree
under `docs/`; nine ADRs recording the decisions taken; correction of every
`[VERIFY]` tag and factual error in the pre-audit draft; and the disabling
of the two inherited deploy workflows
([ADR-0008](../architecture/adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md)) — the
one non-documentation change in the commit.

**Not delivered, deliberately:** no business feature, migration, endpoint,
UI, dependency or generated-file change. The baseline command set was
**not executed** — the Docker daemon was unavailable
([`BASELINE-VERIFICATION.md`](BASELINE-VERIFICATION.md)).

---

## Ready to start

### VS-000a — Frontend test runner (Vitest + Testing Library)

**Status:** Ready · **Depends on:** VS-000

Add Vitest, React Testing Library, jsdom and a `test:unit` script; one
example component test proving the harness works; a CI job; and the
corresponding update to [`TEST-STRATEGY.md`](TEST-STRATEGY.md). `bun run
test` continues to mean Playwright.

This is a **dependency change** and lands as its own small commit, not
inside a feature slice. Decision:
[ADR-0007](../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md).

---

## Blocked

### VS-001 — Organization, Shop and Tenant Ownership

**Status:** Blocked · **Depends on:** VS-000, VS-000a ·
**Spec:** [`slices/VS-001-ORGANIZATION-AND-SHOP.md`](slices/VS-001-ORGANIZATION-AND-SHOP.md)

Blocked by three things, all resolvable:

1. **B-1** — the baseline has never been executed. Start Docker and run
   [`BASELINE-VERIFICATION.md`](BASELINE-VERIFICATION.md) §5, recording
   real results.
2. **B-3** — VS-000a has not landed.
3. **B-7** — two open business questions in the slice spec: what happens to
   shops when an organization is deactivated, and whether the last Owner
   can be removed.

### VS-002 — Product Catalog and Internal SKU

**Status:** Blocked on VS-001 ·
**Spec:** [`slices/VS-002-PRODUCT-CATALOG.md`](slices/VS-002-PRODUCT-CATALOG.md)

Open: SKU code normalisation rules, whether a product may exist without a
variant, whether a SKU code may change after orders reference it, and D-11
(bundles). Also owns the retirement of the template's `Item` entity, in its
own commit, once catalog coverage replaces it.

### VS-003 — Marketplace SKU Mapping

**Status:** Blocked on VS-002 ·
**Spec:** [`slices/VS-003-SKU-MAPPING.md`](slices/VS-003-SKU-MAPPING.md)

Hard gate: **D-06** — is remapping retroactive to already-imported orders?
The answer determines whether the mapping table needs effective dating.

### VS-004 — CSV Import Center

**Status:** Blocked ·
**Spec:** [`slices/VS-004-CSV-IMPORT-CENTER.md`](slices/VS-004-CSV-IMPORT-CENTER.md)

Two hard gates: **B-6 / D-13** (personal-data handling and retention) and
**B-5** (no real marketplace sample files). Also D-09, the idempotency key.

### VS-005 — Import Validation and Error Handling

**Status:** Blocked on VS-004 ·
**Spec:** [`slices/VS-005-IMPORT-VALIDATION.md`](slices/VS-005-IMPORT-VALIDATION.md)

Also owns agreeing the shared exception record shape that VS-013 will
consume (R-18).

### VS-005a — Backend domain-package restructure

**Status:** Blocked on VS-005

Pure refactor, no behaviour or schema change; the regenerated client diff
must be empty. Decision:
[ADR-0009](../architecture/adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md).

### VS-006 — Time-Effective Cost History

**Status:** Blocked on VS-002, VS-005a ·
**Spec:** [`slices/VS-006-COST-HISTORY.md`](slices/VS-006-COST-HISTORY.md)

Hard gates: mandatory questions **#7** (cost basis) and **#8**
(recalculation on correction).

---

## Seeded — spec exists, business analysis incomplete

These have specs at **Seeded** level. Each must be brought to Specified —
with its business questions answered — before it can pass the Definition of
Ready. Filling them in now would mean inventing rules nobody can validate
yet.

| Slice | Spec | Principal open questions |
|---|---|---|
| VS-007 Orders | [spec](slices/VS-007-ORDERS.md) | D-08 timezone/periods, D-09 natural key |
| VS-008 Fee normalization | [spec](slices/VS-008-FEE-NORMALIZATION.md) | Q1, Q2, Q3, fee taxonomy |
| VS-009 Profit engine | [spec](slices/VS-009-PROFIT-ENGINE.md) | The formula itself, Q7, Q8, Q11, D-12 |
| VS-010 Refunds | [spec](slices/VS-010-REFUNDS-AND-RETURNS.md) | Q4, Q5, Q6, Q14 |
| VS-011 Settlement import | [spec](slices/VS-011-SETTLEMENT-IMPORT.md) | Q10, Q12, Q13; blocked by B-5 |
| VS-012 Reconciliation | [spec](slices/VS-012-RECONCILIATION.md) | Q9, Q13, Q14, Q15, tolerance |
| VS-013 Exception management | [spec](slices/VS-013-EXCEPTION-MANAGEMENT.md) | Taxonomy, lifecycle, auto-close |
| VS-014 Dashboard | [spec](slices/VS-014-PROFITABILITY-DASHBOARD.md) | D-08, KPI inclusion rules |
| VS-015 Drill-down | [spec](slices/VS-015-DRILL-DOWN.md) | P-3, P-4 personal data in lineage |
| VS-016 Roles and permissions | [spec](slices/VS-016-ROLES-AND-PERMISSIONS.md) | Q15, Accountant role, shop scoping |
| VS-017 Audit trail | [spec](slices/VS-017-AUDIT-TRAIL.md) | P-7 retention |
| VS-018 Reports and exports | [spec](slices/VS-018-REPORTS-AND-EXPORTS.md) | Report catalogue, P-5 |
| VS-019 Production hardening | [spec](slices/VS-019-PRODUCTION-HARDENING.md) | Retention, RPO/RTO, deploy decision |

---

## Board maintenance

When a slice starts: bring its spec to Specified, confirm the Definition of
Ready, create the branch `slice/VS-0xx-short-name` off **`master`**, and
update its status here. When it finishes: link its integration report
before marking it Done.

Open decisions blocking these slices are tracked in
[`../ba/08-BUSINESS-RULES.md`](../ba/08-BUSINESS-RULES.md); risks and
blockers in [`RISK-REGISTER.md`](RISK-REGISTER.md).
