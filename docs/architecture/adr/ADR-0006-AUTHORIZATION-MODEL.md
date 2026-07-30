# ADR-0006 — Platform superuser and organization Owner are separate concerns

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

The inherited template has exactly one authorization axis: a boolean
`user.is_superuser`, checked by `get_current_active_superuser` in
`backend/app/api/deps.py`. It gates user administration, the test-email
endpoint and the password-recovery HTML endpoint. Everything else is
"any authenticated active user", with ad-hoc `owner_id` comparisons inside
individual handlers.

SellerMargin needs organization-scoped roles: an Owner who runs a seller's
organization, an Analyst who does day-to-day operations, a Viewer who only
reads. The question raised by the VS-000 audit (D-04) was whether the
template's `is_superuser` should be folded into that role model or kept
separate. Two overlapping authorization models applied inconsistently is a
well-known source of privilege-escalation defects.

## Decision

**Keep the two as separate, non-overlapping concerns. Do not collapse them.**

### `is_superuser` — platform administration, outside any organization

- Retained exactly as it is today, including the existing
  `get_current_active_superuser` dependency and the endpoints it gates.
- Its meaning is **operating the SellerMargin installation**: managing user
  accounts, platform diagnostics, support access.
- It is **not** an organization role. A superuser is not implicitly a
  member of any organization.
- **A superuser does not automatically get access to organization business
  data.** Reading a seller's orders, costs or profit requires a membership
  like anyone else. Any support access to tenant data is a separate,
  explicitly-designed, audited capability — not an implicit consequence of
  the flag.
- Superuser status is granted only by another superuser and is auditable.

### `Owner` / `Analyst` / `Viewer` — roles within one organization

- Granted through a `membership` record (user × organization × role).
- `Owner` is the highest role **within one organization**: manage shops,
  members, roles, fee rules, confirm reconciliations, unlock periods.
- A user may hold different roles in different organizations
  simultaneously.
- Organization roles convey no platform privilege whatsoever.

### How they interact

| Question | Answer |
|---|---|
| Does `is_superuser` grant organization data access? | **No** |
| Does Owner grant platform administration? | **No** |
| Can one person hold both? | Yes — they are independent |
| Which is checked first on a business endpoint? | Organization scope + permission. `is_superuser` is not consulted |
| Which is checked on a platform endpoint (`/users/`, `/utils/test-email/`)? | `is_superuser` only |
| Can a superuser create an organization for someone? | Yes — that is platform administration, and it is audited |

The practical rule for implementers: **a business endpoint must never
contain `if current_user.is_superuser`.** If one appears, the endpoint is
mixing the two axes.

## Alternatives considered

- **Collapse into one role hierarchy with a `SuperAdmin` role above Owner** —
  rejected. It makes every business query implicitly bypassable by a
  platform flag, which is precisely the tenant-isolation failure mode the
  project treats as high-impact (R-06). It also makes the isolation tests
  weaker, because the strongest actor is exempt from them.
- **Remove `is_superuser` entirely, model platform admin as an
  organization** — rejected. It would require reworking working template
  endpoints and tests for no product benefit, and platform administration
  genuinely is not organization-scoped.
- **Let `is_superuser` read but not write tenant data** — rejected as an
  implicit rule. If support access to tenant data is needed, it should be
  an explicit, audited, time-bounded capability, not a silent read
  exemption.

## Consequences

- Positive: tenant-isolation tests apply to *every* actor, with no
  exempt superuser path.
- Positive: no change needed to existing template auth code in VS-001;
  the new axis is added alongside it.
- Trade-off: the project owner, as a superuser, cannot inspect a seller's
  data through the UI without a membership. This is intended, and it is a
  point worth making explicitly in a portfolio review.
- Trade-off: two concepts to explain in documentation. Handled by
  [`../../security/AUTHORIZATION-POLICY.md`](../../security/AUTHORIZATION-POLICY.md).
- Obligation: VS-001 adds membership and roles without touching
  `is_superuser` semantics. VS-016 formalises the permission catalogue and
  re-gates endpoints against named permissions.
- Obligation: a test must assert that a superuser without a membership
  receives 404 on organization-scoped endpoints.

## Related

[`ADR-0002-TENANCY-MODEL.md`](ADR-0002-TENANCY-MODEL.md),
[`../../security/AUTHORIZATION-POLICY.md`](../../security/AUTHORIZATION-POLICY.md),
[`../../ba/02-STAKEHOLDER-MAP.md`](../../ba/02-STAKEHOLDER-MAP.md),
mandatory question #15 in [`../../ba/08-BUSINESS-RULES.md`](../../ba/08-BUSINESS-RULES.md).
