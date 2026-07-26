# Screen Inventory

**Status:** Draft — target screens. Section 1 is Verified (what exists
today); everything after it is planned.

Route paths follow the existing TanStack Router file-based convention in
`frontend/src/routes/`. Screens under `_layout` are authenticated.

## 1. Screens that exist today (inherited from the template)

| Route | File | Disposition |
|---|---|---|
| `/login` | `routes/login.tsx` | Keep; rebrand |
| `/signup` | `routes/signup.tsx` | Keep; extend for organization creation (VS-001) |
| `/recover-password` | `routes/recover-password.tsx` | Keep |
| `/reset-password` | `routes/reset-password.tsx` | Keep |
| `/` (dashboard) | `routes/_layout/index.tsx` | **Placeholder** — renders a greeting only. Replaced in VS-014 |
| `/items` | `routes/_layout/items.tsx` | **Retire at VS-002** (ADR-0004) |
| `/settings` | `routes/_layout/settings.tsx` | Keep; extend |
| `/admin` | `routes/_layout/admin.tsx` | Keep as **platform** user admin (superuser only). Distinct from organization member management |

Page titles currently read "FastAPI Template" and are corrected at VS-019.

## 2. Planned screens by slice

### VS-001 — Organization and shops

| Screen | Route | Notes |
|---|---|---|
| Organization switcher | in `AppSidebar` | Not a route; changes active organization for every view |
| Organization settings | `/settings/organization` | Name, currency (read-only after creation) |
| Shops list | `/shops` | `DataTable`; create/edit/deactivate dialogs |
| Members | `/settings/members` | Invite, assign role, remove. Owner-only. **Not** the same as `/admin` |
| First-run empty state | `/` | "No data yet → import your first file" |

### VS-002 — Product catalog

| Screen | Route |
|---|---|
| Products list | `/catalog/products` |
| Product detail (variants, SKUs) | `/catalog/products/$productId` |
| Internal SKU detail | `/catalog/skus/$skuId` — becomes the hub for cost history (VS-006) and SKU profitability (VS-015) |

### VS-003 — SKU mapping

| Screen | Route |
|---|---|
| Mapping workbench (unmapped queue + suggestions) | `/catalog/mappings` |
| Mapping list / history | `/catalog/mappings/history` |
| Mapping history drawer | in-page on SKU detail |

### VS-004 / VS-005 — Import Center

| Screen | Route |
|---|---|
| Import Center home / batch history | `/imports` |
| Import wizard: upload → detect → map columns → preview → confirm | `/imports/new` |
| Batch detail (counts, status) | `/imports/$batchId` |
| Batch errors tab | `/imports/$batchId/errors` |
| Column-mapping profiles | `/imports/profiles` |

The wizard is the most complex new UI in the early roadmap: multi-step,
long-running background work, partial success, and a reprocess loop.

### VS-006 — Cost history

| Screen | Route |
|---|---|
| Cost timeline | on `/catalog/skus/$skuId` |
| Add cost dialog (effective-from) | in-page |
| Missing-cost report | `/costs/missing` |
| Bulk cost upload | reuses the import wizard |

### VS-007 — Orders

| Screen | Route |
|---|---|
| Orders list | `/orders` |
| Order detail (lines + source panel) | `/orders/$orderId` — grows through VS-008, VS-009, VS-010 |

### VS-008 — Fees

| Screen | Route |
|---|---|
| Fee rules admin | `/settings/fee-rules` — Owner-only |
| Fee rule detail with version history | `/settings/fee-rules/$ruleId` |
| Unrecognised fee-label queue | `/imports/unrecognised-fees` |
| Charges breakdown | on order detail |

### VS-009 — Profit

| Screen | Route |
|---|---|
| Profit breakdown panel | on order detail |
| "Explain this number" drawer | shared component, reused by the dashboard |
| Recalculate action | on order / batch views; permissioned |

### VS-010 — Refunds

| Screen | Route |
|---|---|
| Refunds list | `/refunds` |
| Refunds tab | on order detail |

### VS-011 / VS-012 — Settlement and reconciliation

| Screen | Route |
|---|---|
| Statements list | `/settlements` |
| Statement detail | `/settlements/$statementId` |
| Reconciliation workspace (matched / unmatched / variance) | `/reconciliation` |
| Variance detail with dual drill-through | `/reconciliation/$runId/variance/$varianceId` |
| Period lock management | `/settings/periods` |

### VS-013 — Exceptions

| Screen | Route |
|---|---|
| Exception inbox | `/exceptions` |
| Exception detail with drill-through | `/exceptions/$exceptionId` |

### VS-014 / VS-015 — Dashboard and drill-down

| Screen | Route |
|---|---|
| Profitability dashboard (**replaces the placeholder**) | `/` |
| Period drill-down | `/analytics/period` |
| SKU profitability detail | on `/catalog/skus/$skuId` |
| Channel comparison | `/analytics/channels` |

### VS-016 / VS-017 / VS-018 — Permissions, audit, reports

| Screen | Route |
|---|---|
| Permission matrix view | `/settings/roles` |
| Audit log viewer | `/settings/audit` — Owner-only |
| Entity history drawer | in-page on cost, fee rule, mapping, reconciliation |
| Reports index | `/reports` |
| Report viewer with parameters | `/reports/$reportId` |
| Export history | `/reports/exports` |

## 3. Navigation structure (target)

```
Sidebar
├── Dashboard                /
├── Imports                  /imports
├── Catalog
│   ├── Products             /catalog/products
│   ├── SKU Mappings         /catalog/mappings
│   └── Missing Costs        /costs/missing
├── Orders                   /orders
├── Reconciliation           /reconciliation
├── Exceptions               /exceptions          [badge: open count]
├── Reports                  /reports
└── Settings
    ├── Organization         /settings/organization
    ├── Shops                /shops
    ├── Members              /settings/members    [Owner]
    ├── Fee Rules            /settings/fee-rules  [Owner]
    ├── Periods              /settings/periods    [Owner]
    ├── Audit Log            /settings/audit      [Owner]
    └── My Account           /settings

Platform (superuser only, outside organization context)
└── Users                    /admin
```

`/admin` is deliberately separated from organization member management —
the two-axis authorization model in
[`../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md`](../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md)
must be visible in the navigation, not just in the code.

## 4. Cross-cutting components

| Component | Used by | Notes |
|---|---|---|
| Organization switcher | every authenticated screen | VS-001 |
| "Explain this number" drawer | dashboard, orders, SKU detail | VS-009; the traceability primitive |
| Source-row viewer | order detail, drill-down | Personal data masked by default |
| Effective-dated timeline | cost history, fee rules, mappings | One component, three uses |
| Data-health tile | dashboard | Unmapped SKUs, missing costs, open exceptions, unreconciled amount |
| Money display | everywhere | Currency, sign, consistent precision |
| Bulk-action toolbar | mapping workbench, exceptions, import errors | |

## 5. Screen-level Definition of Done

Every screen ships with loading, empty, error and happy states
([`UX-PRINCIPLES.md`](UX-PRINCIPLES.md) §4), keyboard accessibility, correct
behaviour in both themes, permission-aware disabled controls with reasons,
and — once VS-000a lands — component tests for the four states.
