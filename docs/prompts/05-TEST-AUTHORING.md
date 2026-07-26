# Prompt — Test Authoring

Use to write the test layers for a slice, or to close a coverage gap.

---

```
Write the tests for <VS-0xx — Name>.

Read first:
- docs/delivery/TEST-STRATEGY.md
- docs/delivery/slices/<VS-0xx-SLICE-FILE>.md  (its acceptance criteria)
- docs/delivery/BASELINE-VERIFICATION.md
- docs/security/AUTHORIZATION-POLICY.md §6

Environment facts:
- Backend tests are pytest and run against a REAL PostgreSQL database.
  backend/tests/conftest.py seeds via init_db and DELETES all Item and User
  rows at session teardown. Start the DB first: docker compose up -d db
- Run with: cd backend && uv run bash scripts/test.sh
- CI enforces: uv run coverage report --fail-under=90. This is a hard gate
- E2E is Playwright (Chromium only), in frontend/tests/*.spec.ts.
  "bun run test" runs PLAYWRIGHT, not unit tests
- There is no component-test runner until VS-000a (ADR-0007)

Required coverage for this slice:
1. Unit — every business rule and formula this slice introduces or changes,
   at boundary values, not just the happy value
2. Integration — every endpoint added or changed, including validation and
   error paths
3. Authorization — for each endpoint: each allowed role passes, each denied
   role gets 403, another organization gets 404, and a superuser WITHOUT a
   membership gets 404
4. E2E — the slice's primary happy-path journey

Project-specific tests that must exist where the slice touches them:
- Idempotency: re-importing identical data creates no duplicates
- Cost history: adding or correcting a cost does not mutate existing rows,
  and ranges never overlap
- Effective-date resolution: correct record returned at exact boundaries,
  and a missing covering record raises a reportable exception rather than
  defaulting to zero
- Traceability: a calculated figure can be traced to its source rows and
  the rule versions used
- Aggregate/detail invariant: drill-down sums equal the aggregate
- Audit: each auditable action emits exactly one event, and events cannot
  be modified or deleted

Map every test to the acceptance criterion it proves. If an acceptance
criterion has no test, say so explicitly rather than quietly leaving it.

Do not write a test that asserts the implementation back to itself. A unit
test can be green while checking the wrong formula — cross-check against
the worked examples in docs/ba/07-PROFIT-CALCULATION-SPEC.md.
```

---

## Notes

- The last paragraph is the point of the whole prompt. Tests that mirror
  the implementation prove nothing about business correctness, which is
  why UAT is a separate step.
- The teardown warning is worth keeping: pointing the test suite at a
  database with real data destroys it.
