# Integration Report Template

Fill this out at the end of a vertical slice, before requesting human
review. This replaces the original pack's cross-agent integration report
(which recorded a handoff between a backend agent and a frontend agent)
— here it records the single agent's own end-to-end verification instead.

---

**Slice:** VS-0xx — <title>
**Date:** <date>
**Branch:** `slice/VS-0xx-short-name` (off `master`)

## What was built

Short summary of backend, frontend, and data model changes actually
delivered — not the original plan, the actual result.

## Contract verification

- [ ] `bash scripts/generate-client.sh` run; `git diff --exit-code
      frontend/src/client/` is clean (a diff means the committed client was
      stale — report it)
- [ ] The client diff was reviewed; any non-additive change is explained
- [ ] Every new router declares a tag
- [ ] Frontend builds against the regenerated client with no manual patches

Note: `openapi.json` is git-ignored and is a build artifact. The reviewable
artifact is the `frontend/src/client/` diff.

## Test results

Report **actual** numbers, not expected ones. If a step was not run, say
so and say why — never report an expected result as an actual one.

- [ ] Backend tests (`uv run bash scripts/test.sh`): passed / failed / skipped
- [ ] **Coverage (`uv run coverage report --fail-under=90`): actual %**
- [ ] Static analysis (`uv run bash scripts/lint.sh`): result
- [ ] Frontend component tests: result — **or "not applicable, VS-000a has
      not landed"** ([ADR-0007](../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md))
- [ ] E2E (Playwright): passed / failed / skipped
- [ ] Frontend build (`bun run --filter frontend build`): result
- [ ] Every known-failing or skipped test, and why

## Migration verification

- [ ] Migration read line by line against
      [`MIGRATION-AND-ROLLBACK.md`](MIGRATION-AND-ROLLBACK.md) §3 — not
      blindly trusted from autogenerate
- [ ] Downgrade **executed** against seeded data, not merely written
- [ ] Still exactly one head (`uv run alembic heads`)
- [ ] Confirmed no unintended data loss or history rewrite

## Definition of Done

Go through [`DEFINITION-OF-READY-DONE.md`](DEFINITION-OF-READY-DONE.md)
item by item and mark each **met** or **not met**. State the unmet ones
explicitly; do not omit them.

## Traceability check

Demonstrate (don't just assert) that a value produced by this slice can
be traced back to its source row and the rule/version that produced it.

## UAT summary

Link to or summarize the UAT note from
`docs/ba/09-UAT-STRATEGY.md` for this slice — actual vs. expected,
pass/fail, and root cause for any failure.

## Deviations from the spec

Anything that differed from `VERTICAL-SLICE-SPEC-TEMPLATE.md` for this
slice, and why.

## Outstanding items for human review

Anything intentionally left for the human to decide or verify before
merge.
