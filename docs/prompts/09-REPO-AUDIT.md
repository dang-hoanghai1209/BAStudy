# Prompt — Repository Audit

Use periodically to re-verify that the documentation still matches the
repository. Documentation that contradicts the code is worse than none.

Run this at minimum: after every third slice, before any release, and
whenever a document is suspected of being stale.

---

```
Audit this repository against its documentation. Do not implement anything.

Read completely before starting:
- CLAUDE.md
- docs/architecture/CURRENT-STATE.md
- docs/delivery/BASELINE-VERIFICATION.md

Then inspect the ACTUAL repository and report where documentation and
reality have diverged. Use evidence — file paths, line numbers, command
output. Do not guess, and do not assume a document is right because it is
committed.

Check specifically:
1. Backend structure — does it still match CURRENT-STATE.md §3? Has the
   ADR-0009 restructure happened yet?
2. Data model — every table in the database vs. DATA-MODEL.md. Alembic
   head vs. what is documented. Still exactly one head?
3. API surface — every route vs. CURRENT-STATE.md §5 and the permission
   matrix in security/AUTHORIZATION-POLICY.md §4. Is any endpoint missing
   from the matrix?
4. Generated files — is the committed frontend/src/client/ up to date?
     bash scripts/generate-client.sh && git diff --exit-code frontend/src/client/
   A diff means the committed client was stale
5. Tests — actual counts and coverage vs. TEST-STRATEGY.md. Is the 90%
   gate still passing? Has the component-test runner landed (ADR-0007)?
6. Business rules — is any rule implemented in code that is NOT recorded
   in ba/08-BUSINESS-RULES.md? That is the failure mode R-01 exists for
7. Retirement schedule — which ADR-0004 items are now due? Is Item still
   present after VS-002?
8. Open decisions — which of the mandatory questions and the D-series
   decisions are still unanswered, and which slices they now block
9. Deploy workflows — still disabled per ADR-0008?
10. Secrets — are the changethis placeholders still in .env? Report
    variable NAMES and status only. Never reproduce a secret value

Report:
- Confirmed accurate (briefly)
- Diverged: what the document says, what is actually true, which file and
  line
- Documentation that should be updated, and what it should say
- New risks for the register
- Any [VERIFY]-style unverified assumption that has crept back in

Do not edit any file in this pass. Produce the findings first.
```

---

## Notes

- Item 6 is the one that matters most for this project's credibility: a
  business rule that exists only in code is exactly what the whole
  documentation-first workflow is meant to prevent.
- Keep the "do not edit in this pass" instruction. Auditing and fixing in
  one step makes it impossible to review the findings independently.
