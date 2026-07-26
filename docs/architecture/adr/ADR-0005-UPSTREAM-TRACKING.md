# ADR-0005 — Tracking the upstream template via git, not Copier

**Status:** Accepted (mechanism) / Open (whether to keep tracking at all — see §Open question)
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

The pre-audit documentation assumed SellerMargin was a project *generated
from* the Full Stack FastAPI Template. The VS-000 audit found otherwise:

- `git rev-list --count HEAD` → **1392 commits**, all upstream template history
- `copier.yml` (the template *definition*) is present at the repository root
- `.copier/.copier-answers.yml.jinja` exists but **no rendered
  `.copier-answers.yml` does**
- root `package.json` is named `fastapi-full-stack-template`
- no tracked file mentions "SellerMargin"

This is a **fork of the template repository itself**. `copier update`
requires an answers file recording which template version and which answers
produced the project; there is none, so that upgrade path does not exist
and never did.

`origin` originally pointed at
`https://github.com/fastapi/full-stack-fastapi-template.git` and has since
been corrected to `https://github.com/dang-hoanghai1209/BAStudy.git`.

## Decision

**Treat the upstream template as a git remote, not as a Copier template.**

1. `origin` is the project's own repository
   (`dang-hoanghai1209/BAStudy`). All SellerMargin work pushes here.
2. If upstream changes are wanted, add the template as a **separate,
   fetch-only remote**:
   ```bash
   git remote add upstream https://github.com/fastapi/full-stack-fastapi-template.git
   git remote set-url --push upstream DISABLED
   git fetch upstream
   ```
   The `--push DISABLED` guard makes an accidental push to the upstream
   project impossible.
3. Upstream changes are adopted **selectively by cherry-pick**, not by
   merging `upstream/master` wholesale. Security fixes and dependency
   bumps are worth taking; template demo-content changes are not.
4. Every adopted upstream commit is reviewed like any other change — it may
   touch `Item`, the deploy workflows, or other things being retired under
   ADR-0004.
5. `copier.yml`, `.copier/` and `hooks/post_gen_project.py` are retired at
   VS-019. They are inert now but they mislead a reader into believing a
   Copier upgrade path exists.

## Alternatives considered

- **`copier update`** — not available. No answers file. Reconstructing one
  by hand would mean guessing which template commit this fork diverged from
  and which answers were given, then trusting a three-way merge built on
  that guess.
- **Merge `upstream/master` periodically** — rejected. As SellerMargin
  diverges (retiring `Item`, replacing the dashboard, restructuring the
  backend), a full merge produces large conflicts in exactly the files
  being deliberately changed, and risks resurrecting retired content.
- **Cut history entirely** (squash to one commit, drop the upstream
  relationship) — rejected *for now*, but see below. It would make the
  repository unambiguously SellerMargin's, at the cost of losing the
  ability to pull upstream security fixes and losing the provenance that
  explains why the codebase looks the way it does.

## Open question requiring the owner's decision

Whether to keep tracking upstream at all is **not yet settled**. The
trade-off:

| Keep tracking | Cut history |
|---|---|
| Can cherry-pick upstream security and dependency fixes | Must handle all dependency maintenance independently |
| Provenance is visible and honest | `git log` reads as SellerMargin's own work from commit one |
| 1392 unrelated commits dominate the history a reviewer sees | Clean, readable history for a portfolio reviewer |
| Retirement work (ADR-0004) shows up as diffs against a known baseline | Retirement work becomes invisible |

Recommendation: **keep tracking through VS-019**, then reconsider. The
dependency-maintenance benefit is real while the project is still close to
the template; it decays as divergence grows.

## Consequences

- Positive: upstream security fixes remain reachable.
- Positive: no dependence on Copier machinery that cannot work here.
- Trade-off: a portfolio reviewer running `git log` sees mostly upstream
  history until this is revisited. Mitigated by the documentation set
  making the provenance explicit rather than hiding it.
- Obligation: the fetch-only upstream remote must be configured before the
  first upstream cherry-pick, not improvised at the time.
- Obligation: revisit this ADR at VS-019 and either supersede it or
  confirm it.

## Related

[`ADR-0004-TEMPLATE-INHERITANCE.md`](ADR-0004-TEMPLATE-INHERITANCE.md),
[`../CURRENT-STATE.md`](../CURRENT-STATE.md) §1,
[`../../delivery/RISK-REGISTER.md`](../../delivery/RISK-REGISTER.md) R-13.
