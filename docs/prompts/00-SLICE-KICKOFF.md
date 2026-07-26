# Prompt — Slice Kickoff

Use at the start of a vertical slice, after its spec exists and its
Definition of Ready passes.

---

```
Implement vertical slice <VS-0xx — Name> for SellerMargin.

Read these first, completely, before writing any code:
- CLAUDE.md
- docs/delivery/slices/<VS-0xx-SLICE-FILE>.md
- docs/ba/08-BUSINESS-RULES.md
- docs/architecture/DATA-MODEL.md
- docs/architecture/CURRENT-STATE.md
- docs/security/AUTHORIZATION-POLICY.md
- docs/delivery/DEFINITION-OF-READY-DONE.md

Before writing code, confirm the Definition of Ready actually passes for
this slice. If any business rule this slice depends on is unanswered in
docs/ba/08-BUSINESS-RULES.md, STOP and tell me which one. Do not assume an
answer and do not code around the gap.

Work in this order and do not reorder or skip a step:
1. Business rules and acceptance criteria confirmed present
2. Data model change + migration (read the generated migration; verify it
   against docs/delivery/MIGRATION-AND-ROLLBACK.md §3)
3. Backend implementation
4. Regenerate the client: bash scripts/generate-client.sh — never hand-edit
   frontend/src/client/** or routeTree.gen.ts
5. Frontend implementation, including loading / empty / error states
6. Tests per docs/delivery/TEST-STRATEGY.md
7. Integration report from docs/delivery/INTEGRATION-REPORT-TEMPLATE.md

Constraints:
- Branch: slice/<VS-0xx-short-name> off master (the default branch is
  master, not main)
- Every new business table carries organization_id; shop_id where the
  record is shop-specific
- Money is Decimal / NUMERIC(18,4). Never float, at any layer
- Nothing overwrites a value with history; append and supersede
- Do not change dependencies, lockfiles, .env or Docker configuration
- Stay inside the slice's scope. If something looks like it belongs to
  another slice, say so rather than absorbing it

Definition of Done is docs/delivery/DEFINITION-OF-READY-DONE.md. A slice is
not done on green tests alone.

Start by restating the scope in your own words and listing what you will
build, so I can correct a misunderstanding before code exists.
```

---

## Notes

- The final instruction — restate scope before building — catches
  misreadings cheaply. Do not remove it to save a round trip.
- If the slice touches a mandatory business question, name that question
  number explicitly in the prompt rather than trusting it to be noticed.
