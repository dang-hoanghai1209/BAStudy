# ADR-0009 — Stay flat until VS-006, then restructure into domain packages

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

The pre-audit architecture document presented `backend/app/modules/` as
though it existed. The VS-000 audit found it does not. The real backend is
flat:

```
backend/app/
├── models.py     ALL models (User, Item)
├── crud.py       ALL data access
├── api/routes/   items.py, login.py, private.py, users.py, utils.py
└── core/         config.py, db.py, security.py
```

`ADR-0001` commits the project to a modular monolith, and
[`../TARGET-ARCHITECTURE.md`](../TARGET-ARCHITECTURE.md) §2 describes the
target domain-package layout. The open question is *when* to move, and
whether the move happens inside a feature slice.

A single `models.py` will not survive this domain. By VS-009 it would hold
organizations, shops, products, variants, SKUs, mappings, import batches,
source rows, errors, orders, order lines, cost history, fee rules, charges
and calculation ledger entries — well over a thousand lines with no
boundaries, and SQLModel relationship resolution becomes fragile as it
grows.

Moving too early has its own cost: with two template entities and no
domain, package boundaries would be guessed rather than observed.

## Decision

**Keep the flat structure through VS-005. Restructure into domain packages
as a dedicated refactor slice (VS-005a) before VS-006 begins.**

Rationale for that trigger point: by the end of VS-005 the project has
organizations, shops, catalog, mapping and imports — five real domains with
observed boundaries, roughly a dozen entities, and enough test coverage to
make a pure-refactor safe. VS-006 onward (costs, fees, profit) is where
the code volume grows fastest, so the move pays off immediately after.

### Rules for the restructure

- It is a **pure refactor**: moves, imports, and `__init__` exports only.
  **No behaviour change, no schema change, no migration, no API change.**
- The generated OpenAPI schema must be **byte-identical** before and after
  (regenerate the client; a non-empty diff in `frontend/src/client/**`
  means behaviour changed and the refactor is wrong).
- Alembic `env.py` must still import every model. A single
  `app/domains/__init__.py` that re-exports all models keeps metadata
  registration reliable and prevents "table not found in autogenerate".
- Tests move with their domain and must pass unchanged.
- It lands as its own commit with no feature work attached.

### Rules until then

- New entities go into `models.py`, but **grouped and section-commented by
  domain** so the eventual split is mechanical rather than archaeological.
- New data access goes into `crud.py` with the same grouping, or into a
  domain-named module where that is already natural.
- New routers go into `api/routes/<domain>.py` — one file per domain from
  the start, since routers are already split that way.
- **Every router declares `tags=[...]`.** The generated client's service
  names derive from the first tag (see
  [`../GENERATED-FILES.md`](../GENERATED-FILES.md) §3), so tags are part of
  the public contract, not decoration.

## Alternatives considered

- **Restructure now, in VS-000/VS-001** — rejected. Boundaries would be
  invented from a document rather than derived from real code, and it
  would put a large structural diff in front of the first business slice
  for no product benefit.
- **Never restructure; keep flat** — rejected. It contradicts ADR-0001,
  and a 1500-line `models.py` is a maintainability and review problem, not
  a style preference.
- **Restructure incrementally, one domain at a time as each slice lands** —
  rejected. Long-lived mixed structure is harder to reason about than
  either endpoint, and every slice would carry an unrelated structural
  diff, making feature review noisier.
- **Split only `models.py`, leave routes and crud flat** — rejected as a
  half-measure that still leaves data access unbounded.

## Consequences

- Positive: early slices stay small and reviewable; no structural churn
  competing with business logic.
- Positive: the eventual split is informed by real boundaries.
- Trade-off: `models.py` and `crud.py` get uncomfortably large around
  VS-004–VS-005. Accepted, bounded, and mitigated by section grouping.
- Trade-off: a refactor slice produces no user-visible value. It is
  scheduled and named so it is not mistaken for scope creep inside a
  feature slice.
- Obligation: VS-005a must be on the task board as a real slice with its
  own spec, not an implicit intention.
- Risk: a pure refactor that accidentally changes the API. Mitigated by the
  byte-identical-schema check above.

## Related

[`ADR-0001-MODULAR-MONOLITH.md`](ADR-0001-MODULAR-MONOLITH.md),
[`../TARGET-ARCHITECTURE.md`](../TARGET-ARCHITECTURE.md) §2,
[`../CURRENT-STATE.md`](../CURRENT-STATE.md) §3,
[`../../delivery/VERTICAL-SLICE-ROADMAP.md`](../../delivery/VERTICAL-SLICE-ROADMAP.md).
