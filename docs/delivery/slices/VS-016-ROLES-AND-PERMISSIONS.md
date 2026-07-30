# VS-016 — Roles and Permissions (RBAC hardening)

**Slice ID:** VS-016 · **Depends on:** VS-001, VS-012 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.

## Business objective

Move from the coarse roles seeded in VS-001 to an explicit, enforced and
auditable permission model covering every sensitive action the system has
accumulated by this point.

## BA learning outcome

Writing a permission matrix as a **business document** — who may do what,
and why — and then proving enforcement rather than asserting it.

## Business rules required — **pending**

| Question | Status |
|---|---|
| **Q15** — who may edit a fee rule and confirm a reconciliation | Draft in [authorization policy](../../security/AUTHORIZATION-POLICY.md); needs confirmation |
| Is "Accountant" a distinct role, or Viewer + export rights? | **Open** — raised in [stakeholder map](../../ba/02-STAKEHOLDER-MAP.md) |
| Per-shop vs. per-organization role scoping | **Open** — `membership.shop_id` exists from VS-001 but is unused |
| Fixed roles, or customer-defined custom roles? | **Open** — recommend fixed |
| **P-5** — who may export data containing personal data | **Open** |

The relationship between `is_superuser` and organization Owner is **already
decided** and is not reopened here
([ADR-0006](../../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md)).

## Entities involved

`role`, `permission`, `role_permission`, plus use of the optional
`membership.shop_id` scoping. Refactors — does not duplicate — the existing
`get_current_active_superuser` dependency.

## Expected API capabilities

Permission catalogue. Role management. Assign a role to a member,
optionally scoped to a shop. Effective permissions for the current user.
**Every existing endpoint re-gated against named permissions** rather than
ad-hoc role checks.

## Expected frontend screens

Members and roles admin; permission matrix view (`/settings/roles`); UI
affordances disabled with a reason per permission; clear "you don't have
permission" states.

## Acceptance criteria

1. The documented matrix in
   [`../../security/AUTHORIZATION-POLICY.md`](../../security/AUTHORIZATION-POLICY.md)
   §4 is complete, and **every cell has a passing test**.
2. Sensitive actions are permission-gated server-side: edit fee rule,
   correct a past cost, confirm reconciliation, unlock a period,
   recalculate a reported period, soft-delete an import batch, change a
   member's role.
3. Removing a permission takes effect on the caller's next request, with no
   stale-permission window.
4. UI hiding never substitutes for server enforcement — proven by a test
   that calls the API directly with an insufficient role.
5. A superuser without a membership still receives 404 on every
   organization-scoped endpoint.
6. Permission changes are audited with a reason.

## Test layers

Unit (permission resolution, shop-scoped roles); **integration — a
matrix-driven test over every endpoint × every role**, generated from the
documented matrix so a new endpoint without a matrix row fails the suite;
contract; frontend (disabled-with-reason states); E2E (a Viewer cannot see
or invoke a restricted action).

## Dependencies

VS-001. Realistically scheduled after VS-012, so that the full set of
sensitive actions is known rather than guessed.

## Risks

| Risk | Mitigation |
|---|---|
| Retro-fitting permissions across fifteen slices' worth of endpoints is large and easy to leave partially applied | Matrix-driven test that fails when an endpoint has no matrix row |
| The binary superuser model fights the role model | Already resolved by ADR-0006; do not reopen |
| Permission checks cached and going stale | Acceptance criterion 3; no cross-request caching |

## CV evidence

A permission matrix as a maintained business document, backed by a
matrix-driven test suite; explicit separation of UI affordance from server
authorization — a distinction many projects blur.

## Explicit exclusions

No SSO, SAML or OIDC. No attribute-based access control. No field-level
permissions. No delegated cross-organization administration. No
time-bounded support access. No customer-defined custom roles.
