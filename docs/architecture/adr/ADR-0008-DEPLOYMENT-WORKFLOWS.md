# ADR-0008 — Disable the inherited deploy workflows until real infrastructure exists

**Status:** Accepted and applied in VS-000
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

The template ships two deployment workflows:

- `.github/workflows/deploy-staging.yml` — triggers on **every push to
  `master`**
- `.github/workflows/deploy-production.yml` — triggers when a GitHub
  **release is published**

Both run `docker compose up -d` against **self-hosted runners** labelled
`staging` and `production`, and both consume repository secrets
(`SECRET_KEY`, `POSTGRES_PASSWORD`, `FIRST_SUPERUSER_PASSWORD`, SMTP
credentials, `SENTRY_DSN`).

Their only guard is:

```yaml
if: github.repository_owner != 'fastapi'
```

That condition exists to stop the upstream template repository from
deploying itself. On a personal fork it **evaluates to true**, so the jobs
are live.

No self-hosted runner exists for this project. The observable outcome of a
push to `master` would be a queued job that never picks up, or a failing
job on every push — permanently red CI, training everyone to ignore it. The
worse outcome, if a runner is ever registered with those labels for an
unrelated reason, is an unintended deployment using placeholder secrets:
the audit confirmed `.env` still contains three literal `changethis`
values.

This was flagged as risk R-12 during VS-000.

## Decision

**Disable both workflows now, in the VS-000 commit, by neutralising their
triggers.**

- The `on:` trigger of each workflow is commented out and replaced with
  `workflow_dispatch` only, so the workflow can still be invoked manually
  and deliberately once infrastructure exists.
- A header comment in each file explains why it is disabled and points at
  this ADR.
- Job steps, secrets wiring and compose commands are left **unchanged**, so
  re-enabling is a one-line edit rather than a reconstruction.

This is the one non-documentation change in the VS-000 commit. It is
included because it is infrastructure safety, not documentation, and
leaving it for later means every intervening push carries the risk.

### Re-enabling

At VS-019 (production hardening), and only when all of the following hold:

1. A real deployment target exists (self-hosted runner, or the workflows
   are rewritten for a different target).
2. Every `changethis` placeholder is replaced, and the non-local secret
   guard in `backend/app/core/config.py` passes for that environment.
3. `PROJECT_NAME`, `STACK_NAME` and `DOMAIN` are set to SellerMargin values.
4. Backup **and a rehearsed restore** are documented per
   [`../../delivery/MIGRATION-AND-ROLLBACK.md`](../../delivery/MIGRATION-AND-ROLLBACK.md).
5. A rollback procedure covering both application and migration exists.

## Alternatives considered

- **Delete both workflow files** — rejected. They encode a working
  compose-based deployment shape that is worth keeping as a starting point;
  deleting means rewriting from scratch later.
- **Leave them live and let them fail** — rejected. Permanently red CI
  hides real failures, and the accidental-deployment path, however
  unlikely, uses placeholder secrets.
- **Change the guard to a different condition (e.g. a branch or secret
  check)** — rejected as too subtle. A commented-out trigger with an
  explanatory header is unambiguous to the next reader; a clever guard
  condition is not.
- **Move them out of `.github/workflows/`** — rejected. It hides them from
  `zizmor` and from anyone looking for the deployment story in the obvious
  place.

## Consequences

- Positive: pushing to `master` no longer triggers a doomed or unintended
  deployment.
- Positive: CI stays meaningful — a red build means something is actually
  wrong.
- Positive: re-enabling is trivial and the intent is documented in-file.
- Trade-off: nothing deploys automatically. Correct for now; there is
  nothing to deploy and nowhere to deploy it.
- Obligation: VS-019 owns re-enabling or replacing these, against the
  checklist above.

## Related

[`ADR-0004-TEMPLATE-INHERITANCE.md`](ADR-0004-TEMPLATE-INHERITANCE.md),
[`../../delivery/RISK-REGISTER.md`](../../delivery/RISK-REGISTER.md) R-12,
[`../../delivery/slices/VS-019-PRODUCTION-HARDENING.md`](../../delivery/slices/VS-019-PRODUCTION-HARDENING.md),
[`../CURRENT-STATE.md`](../CURRENT-STATE.md) §11.
