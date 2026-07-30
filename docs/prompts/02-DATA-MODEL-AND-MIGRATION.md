# Prompt — Data Model and Migration

Use when a slice adds or changes entities.

---

```
Design and implement the data model change for <VS-0xx — Name>.

Read first:
- docs/architecture/DATA-MODEL.md
- docs/architecture/CURRENT-STATE.md  (the backend is FLAT: all models are
  in backend/app/models.py; there is no app/modules/ directory)
- docs/architecture/HISTORICAL-DATA-POLICY.md
- docs/architecture/adr/ADR-0002-TENANCY-MODEL.md
- docs/architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md
- docs/delivery/MIGRATION-AND-ROLLBACK.md

Entities to add or change: <list>

Non-negotiable constraints:
- organization_id on every business table: non-null, indexed, foreign key
- shop_id additionally where the record is shop-specific, with a constraint
  keeping it consistent with the shop's organization
- Money: NUMERIC(18,4) in the database, Decimal in Python. Never float
- Every monetary record stores its currency code explicitly
- Timestamps: DateTime(timezone=True), stored UTC
- NO ON DELETE CASCADE on business data. Deactivate or supersede instead
- Effective-dated entities (cost, fee rule): effective_from inclusive,
  effective_to exclusive and nullable, with a non-overlap constraint
- Add models to backend/app/models.py grouped and section-commented by
  domain, so the ADR-0009 split later is mechanical

Steps:
1. Propose the schema — columns, types, nullability, keys, indexes,
   constraints — and WAIT for my approval before generating anything.
2. After approval: implement the SQLModel models.
3. Generate the migration:
     docker compose up -d db
     cd backend && uv run alembic revision --autogenerate -m "<message>"
4. READ the generated migration and walk me through it line by line
   against the checklist in docs/delivery/MIGRATION-AND-ROLLBACK.md §3.
   Autogenerate is a candidate, not a correct migration.
5. Prove the downgrade works against seeded data:
     uv run alembic upgrade head && uv run alembic downgrade -1 && uv run alembic upgrade head
6. Confirm there is still exactly one head: uv run alembic heads
7. Update docs/architecture/DATA-MODEL.md in the same change.

Do NOT edit any existing migration in backend/app/alembic/versions/. They
are merged history. Fix forward.
```

---

## Notes

- Step 1's approval gate matters: a schema mistake is far cheaper to fix
  before a migration exists.
- Step 4 is where autogenerate defects are actually caught — type changes
  rendered as drop-and-recreate, missing indexes, missing backfills.
- If autogenerate produces an empty migration, a model is not reachable
  from `app/models.py`. Fix the import, do not hand-write the migration.
