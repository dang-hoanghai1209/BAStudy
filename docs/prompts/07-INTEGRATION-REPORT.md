# Prompt — Integration Report

Use at the end of a slice, before requesting human review.

---

```
Produce the integration report for <VS-0xx — Name>.

Use docs/delivery/INTEGRATION-REPORT-TEMPLATE.md as the structure.

Read first:
- docs/delivery/slices/<VS-0xx-SLICE-FILE>.md
- docs/delivery/DEFINITION-OF-READY-DONE.md
- docs/delivery/BASELINE-VERIFICATION.md

Run and report the ACTUAL result of each — not the expected result:

  docker compose up -d db mailcatcher
  cd backend && uv run bash scripts/test.sh
  uv run coverage report --fail-under=90
  cd backend && uv run bash scripts/lint.sh
  bash scripts/generate-client.sh && git diff --exit-code frontend/src/client/
  bun run --filter frontend build
  docker compose run --rm playwright bunx playwright test

Report:
1. What was actually built — the result, not the plan
2. Contract verification: client regenerated, no hand edits, frontend
   builds against it. If step 5 above produced a diff, that means the
   committed client was stale — say so
3. Test results with real numbers: passed, failed, skipped, coverage
   percentage. Name every skipped or failing test and why
4. Migration verification: what the migration does, that it was read line
   by line, that the downgrade was executed against seeded data, and that
   no history is rewritten
5. Traceability: DEMONSTRATE, do not assert. Show one value and the path
   from it to its source rows and rule versions
6. UAT summary: link the note, state pass/fail and root cause of failures
7. Deviations from the slice spec, and why
8. Definition of Done: go through it item by item and mark each met or not
   met. An unmet item is stated, not omitted
9. Outstanding items for human decision before merge

Rules:
- If something failed, report it with the output. Do not summarise a
  failure as "minor"
- If you did not run a step, say you did not run it and why. Do not report
  an expected result as an actual one
- Note explicitly which Definition of Done items could not be met because
  the component-test runner does not exist yet (ADR-0007)
```

---

## Notes

- The distinction between "did not run" and "ran and passed" is the single
  most important honesty property of this report. The VS-000 baseline is
  the precedent: it reported that nothing could be executed because Docker
  was down, rather than assuming green.
- Item 5 says *demonstrate*. An assertion that traceability works is not
  evidence that it works.
