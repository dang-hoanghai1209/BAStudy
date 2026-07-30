# Migration and Rollback

**Status:** Decided (review process) / Draft (production rollback, which is
finalised in VS-019 when a deployment target exists).

## 1. Current migration state

Alembic, configured with `script_location = app/alembic` in
`backend/alembic.ini`. The database URL is injected at runtime by
`app/alembic/env.py` from settings — it is **not** stored in the ini file.

The revision chain is linear with a single head:

```
e2412789c190  initialize models                      (base)
  → 9c0a54914c78  add max length for string/varchar
    → d98dd8ec85a3  replace id integers in all models
      → 1a31ce608336  add cascade delete relationships
        → fe56fa70289e  add created_at to user and item   ← HEAD
```

The first SellerMargin migration (VS-001) has `down_revision = 'fe56fa70289e'`.

Migrations run through `backend/scripts/prestart.sh`, which waits for the
database, runs `alembic upgrade head`, then seeds the first superuser via
`app/initial_data.py`. This is the same path used locally, in CI
(`test-backend.yml`) and by the compose `prestart` service.

## 2. Creating a migration

```bash
docker compose up -d db                       # a live DB is required
cd backend
uv run alembic revision --autogenerate -m "add organization and shop"
# READ THE GENERATED FILE. Then:
uv run alembic upgrade head
uv run alembic downgrade -1                   # prove the downgrade works
uv run alembic upgrade head
```

Note: `--autogenerate` compares against `app.models` metadata. While the
backend is flat (per ADR-0009), every model must be reachable from
`app/models.py` or autogenerate will silently omit it — producing a
migration that looks clean and is wrong.

## 3. Review checklist — mandatory before committing any migration

Autogenerate produces a **candidate**, not a correct migration. Never
commit one unread. Check every line:

- [ ] **Read the whole file.** Every operation is intentional and
      understood.
- [ ] **No unintended `drop_table` / `drop_column`.** Autogenerate drops
      anything it does not see in the models — including tables another
      branch added.
- [ ] **Type changes are not silent drop-and-recreate.** These lose data.
      Use `alter_column` with an explicit `postgresql_using` cast, or a
      three-step add/backfill/drop.
- [ ] **New non-null columns have a server default or a backfill.** Adding
      `NOT NULL` without one fails on any non-empty table.
- [ ] **`organization_id` is present, non-null and indexed** on every new
      business table (ADR-0002).
- [ ] **`shop_id` is present** where the table is shop-specific, with the
      composite consistency constraint against the shop's organization.
- [ ] **Money columns are `NUMERIC(18,4)`**, never `float`/`double`
      (ADR-0003).
- [ ] **Timestamps are timezone-aware** (`DateTime(timezone=True)`).
- [ ] **Effective-dated tables** have their overlap constraint / index, per
      [`../architecture/HISTORICAL-DATA-POLICY.md`](../architecture/HISTORICAL-DATA-POLICY.md).
- [ ] **No `ON DELETE CASCADE` on business data.** The template uses it for
      `item.owner_id`; it must not be copied onto SellerMargin tables,
      which deactivate rather than delete.
- [ ] **Indexes exist** for every foreign key and every column the slice
      filters on.
- [ ] **`downgrade()` is real**, not `pass`, and was actually executed
      against a database with data in it.
- [ ] **Data migrations are separate** from schema migrations, and are
      idempotent.
- [ ] **`down_revision` points at the current head**, and the chain still
      has exactly one head (`uv run alembic heads`).

## 4. Rules

1. **Never edit a merged migration.** Once it is on `master`, it is
   history. Fix forward with a new revision.
2. **One head, always.** Two heads mean two branches created migrations
   independently. Resolve by rebasing and renumbering `down_revision`, not
   by merging heads, until the project genuinely needs branched migrations.
3. **Destructive changes need explicit human sign-off** recorded in the
   pull request description — dropping a column or table, changing a type
   with possible loss, or any data migration that cannot be reversed.
4. **A migration that rewrites calculation history is refused** unless it
   is the explicit purpose of the change and is signed off. See the
   historical-data policy.
5. **Test against data.** An empty database hides most migration defects.
   Seed representative rows, then migrate.

## 5. Rollback

### Local development

```bash
cd backend
uv run alembic downgrade -1
# nuclear option — destroys all local data:
docker compose down -v && docker compose up -d db && uv run bash scripts/prestart.sh
```

### Deployed environments

**Not yet applicable** — both deploy workflows are disabled
([`../architecture/adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md`](../architecture/adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md))
and no environment exists. The procedure below is the intended one and is
finalised and **rehearsed** in VS-019.

Intended order:

1. **Take a backup first.** Always, before any migration, no exceptions.
2. **Roll back the application** to the previous image. Because migrations
   are additive by preference, the previous application version should
   still run against the new schema.
3. **Roll back the schema only if necessary**, using the tested
   `downgrade()`. A downgrade that has never been executed is not a
   rollback plan.
4. **Restore from backup** if the downgrade cannot preserve data. This is
   the last resort and it is the reason the restore drill is a VS-019
   acceptance criterion.

### Preferring forward-compatible migrations

Design migrations so the previous application version keeps working:

- add columns as nullable, backfill, then enforce non-null in a later
  release;
- add new tables rather than restructuring existing ones;
- rename by add → dual-write → backfill → drop, across releases, never in
  a single migration.

This makes step 2 above sufficient in almost every case, which is the
whole point.

## 6. Backup and restore

**Open — owned by VS-019.** Required before any real data exists:

- backup frequency and retention;
- acceptable data loss (RPO) and downtime (RTO);
- **a rehearsed restore**, not just a backup script. A backup that has
  never been restored is an assumption, not a safeguard.

For local development, `docker compose down -v` destroys the `app-db-data`
volume. There is no backup of local data and none is needed — but be
deliberate about running it.

## 7. Migration-related tests

Per [`TEST-STRATEGY.md`](TEST-STRATEGY.md), every slice with a migration
must show:

- CI runs `alembic upgrade head` from scratch on every build
  (`test-backend.yml` already does this via `prestart.sh`);
- the downgrade was executed manually against seeded data;
- no existing test broke, and coverage still passes the 90% gate;
- for effective-dated tables, the overlap constraint is proven by a test
  that attempts an overlap and expects rejection.
