# Definition of Ready / Definition of Done

**Status:** Decided. Updated at VS-000 with audit findings.

## Definition of Ready (before coding starts on a slice)

A slice is ready when:

- [ ] Relevant business rules exist in
      [`../ba/08-BUSINESS-RULES.md`](../ba/08-BUSINESS-RULES.md) — not
      to-be-decided-while-coding
- [ ] Every mandatory business question **and D-series decision** the slice
      touches is answered and recorded (§1 and §2 of that file)
- [ ] User stories with acceptance criteria exist for the slice
- [ ] The data model impact is drafted in
      [`../architecture/DATA-MODEL.md`](../architecture/DATA-MODEL.md)
- [ ] The slice's spec in [`slices/`](slices/) is at **Specified** level,
      not Seeded
- [ ] Dependencies on prior slices (see
      [`VERTICAL-SLICE-ROADMAP.md`](VERTICAL-SLICE-ROADMAP.md)) are
      actually complete, not just started
- [ ] The baseline command set passes — see
      [`BASELINE-VERIFICATION.md`](BASELINE-VERIFICATION.md) §5. A slice
      cannot start on an unverified baseline

If any box is unchecked, the slice is not ready — produce the missing
artifact first (see CLAUDE.md §4).

## Definition of Done (before a slice is considered complete)

A slice is done when all of the following are true — a slice is **not**
done on green tests alone:

- [ ] **BA artifact** — business rules and acceptance criteria are
      finalized and match what was actually built
- [ ] **Database impact** — migration exists, was manually reviewed (not
      blindly trusted from autogenerate), and doesn't silently rewrite
      history
- [ ] **API** — endpoint(s) implemented; every router declares a tag;
      permission rows added to
      [`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md) §4
- [ ] **Generated client** — regenerated via `bash scripts/generate-client.sh`,
      not hand-edited; the diff was reviewed and any non-additive change
      explained
- [ ] **UI** — screen(s) implemented, including loading, empty and error
      states
- [ ] **Tests** — unit tests for every new/changed rule or formula;
      integration tests for new/changed endpoints; the authorization matrix
      including cross-organization 404 and superuser-without-membership 404;
      frontend tests for new UI states; E2E for the primary happy path
      (see [`TEST-STRATEGY.md`](TEST-STRATEGY.md))
- [ ] **Coverage** — `uv run coverage report --fail-under=90` passes. This
      is CI's gate, not a target
- [ ] **Audit** — every auditable action this slice introduces emits its
      event ([`../security/AUDIT-POLICY.md`](../security/AUDIT-POLICY.md))
- [ ] **Traceability** — a value produced by this slice can be traced back
      to its source data and the rule version that produced it, where
      applicable
- [ ] **Acceptance evidence** — a UAT note per
      [`../ba/09-UAT-STRATEGY.md`](../ba/09-UAT-STRATEGY.md), showing
      actual vs. expected
- [ ] **Documentation** — business rules, data model, KPI and glossary
      entries updated in this same PR
- [ ] **Integration report** — produced using
      [`INTEGRATION-REPORT-TEMPLATE.md`](INTEGRATION-REPORT-TEMPLATE.md)
- [ ] **Human review** — reviewed and approved by the human owner before
      merge to **`master`** (the default branch; there is no `main`)

## Temporary exception

Until **VS-000a** lands, the repository has no frontend component-test
runner ([ADR-0007](../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md)).
Until then the "frontend tests for new UI states" item **cannot be met**,
and the integration report must say so plainly rather than marking it
complete. This is the only sanctioned exception, and it expires when
VS-000a merges.

## Note on this replacing the original DoR/DoD

The original coordination pack assumed a frontend hand-off between two
agents as part of "done". That check is removed; everything else is
retained and adapted for a single agent completing the full slice, plus the
coverage, audit, documentation and baseline items added at VS-000.
