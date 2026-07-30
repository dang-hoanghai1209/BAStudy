# VS-001 — Organization, Shop and Tenant Ownership

**Slice ID:** VS-001
**Depends on:** VS-000, VS-000a
**Status:** Not started — **blocked**, see §12
**Level:** Specified

## Business objective

Establish the tenant boundary so that every record SellerMargin adds
afterwards belongs to exactly one organization, and a user in one
organization can never read another's data — even by guessing an
identifier.

This is foundational rather than valuable in itself. It is first because
every later entity carries `organization_id`, and retrofitting tenancy is
a full-schema migration.

## BA learning outcome

Modelling ownership and access as a **business constraint** — who owns the
data, who may act on it — rather than as a technical afterthought. Producing
a first permission matrix and proving it with tests.

## Business rules required

| Rule | Status |
|---|---|
| An organization is the tenancy boundary; all business data belongs to one | Decided — [ADR-0002](../../architecture/adr/ADR-0002-TENANCY-MODEL.md) |
| A shop belongs to exactly one organization | Decided |
| A user reaches organization data only through a membership | Decided |
| Access is deny-by-default, enforced server-side on every request | Decided |
| Cross-organization access returns 404, not 403 | Decided |
| `is_superuser` grants **no** organization data access | Decided — [ADR-0006](../../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md) |
| Organization currency is set at creation and immutable | Decided — [ADR-0003](../../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md) |
| What happens to shops when an organization is deactivated | **Open** — must be answered before coding |
| Can the last Owner of an organization be removed? | **Open** |
| Can a user belong to more than one organization? | Decided — yes, with independent roles |

## Entities involved

**New:** `organization` (name, currency, status, timestamps),
`shop` (organization_id, marketplace, name, external identifier, status),
`membership` (user_id, organization_id, role, optional shop_id, status),
`marketplace` (reference list), `audit_event` (see below).

**Extended:** `user` — unchanged in structure; gains membership
relationships. `is_superuser` semantics are **not** touched.

**Audit:** the `audit_event` table and its emit helper ship here, not in
VS-017. Every later slice emits into it. Back-filling emission into a dozen
shipped slices is the expensive path — see
[`../../security/AUDIT-POLICY.md`](../../security/AUDIT-POLICY.md) §5.

## Expected API capabilities

| Capability | Permission |
|---|---|
| Create organization (name, currency) | Any authenticated user; creator becomes Owner |
| Get / update organization | Owner (update), all members (read) |
| List / create / update / deactivate shop | Owner (write), all members (read) |
| List members | Owner, Analyst |
| Invite member, assign role, remove member | Owner |
| `GET /users/me` extended with memberships and active organization | Self |

Implementation notes:

- A shared organization-scoping dependency in `app/api/deps.py`, used by
  every business endpoint. Not per-handler filtering.
- Router tag `organizations` / `shops` — the tag determines the generated
  client's service class name
  ([`../../architecture/GENERATED-FILES.md`](../../architecture/GENERATED-FILES.md) §3).
- Existing `/users`, `/login`, `/utils` endpoints are unchanged.

## Expected frontend screens

| Screen | Route |
|---|---|
| Organization switcher | in `AppSidebar` (not a route) |
| Organization settings | `/settings/organization` |
| Shops list + create/edit/deactivate | `/shops` |
| Members management | `/settings/members` (Owner-only) |
| First-run empty state | `/` — "no data yet, import your first file" |

`/admin` stays as platform user administration and is **not** merged with
member management. The two-axis model must be visible in the navigation.

## Acceptance criteria

1. A new user with no membership sees an empty state, not another tenant's data.
2. Requesting a shop belonging to another organization returns **404**, not 403.
3. An Owner can create a shop; an Analyst and a Viewer cannot (403).
4. Switching the active organization changes every list on screen.
5. A superuser **without a membership** receives 404 on every
   organization-scoped endpoint.
6. Organization currency cannot be changed after creation.
7. Deactivating a shop hides it from new selections but leaves it
   resolvable on existing records.
8. Role changes and member removals emit audit events with a reason.
9. A role change takes effect on the caller's next request, with no
   re-login needed and no stale-permission window.

## Test layers

- **Unit** — membership resolution; role checks; currency immutability.
- **Integration** — matrix over every new endpoint × {Owner, Analyst,
  Viewer, non-member, superuser-without-membership}; cross-organization
  fetch by guessed UUID returns 404; audit event emitted exactly once per
  auditable action.
- **Contract** — client regenerated, diff reviewed and committed.
- **Frontend (component)** — loading / empty / error states for the shops
  list and members screens. Requires VS-000a.
- **E2E** — sign up → create organization → create shop → invite a member.

The 90% coverage gate applies. This slice adds a lot of code; plan for
coverage rather than discovering it at CI.

## Dependencies

VS-000 (this commit). **VS-000a** — without the component-test runner, this
slice cannot satisfy the frontend-test line of the Definition of Done.

## Risks

| Risk | Mitigation |
|---|---|
| Tenant scoping applied per-handler drifts and leaves a gap | One shared dependency; matrix-driven isolation tests over *every* endpoint |
| The two authorization axes get mixed and `is_superuser` becomes a data backdoor | ADR-0006; explicit test that a superuser without a membership gets 404 |
| Slice grows: catalog, imports and full RBAC all feel adjacent | Exclusions below are binding; permission *granularity* is VS-016 |
| Audit table designed too narrowly for later slices | Event shape fixed in AUDIT-POLICY.md §2 before coding |
| `organization_id` / `shop_id` consistency drifts | Composite FK or write-time constraint, plus a test |

## CV evidence

A permission matrix as a business document, backed by a matrix-driven test
suite. A tenant-isolation test suite that treats the most privileged actor
as no exception. An ADR trail explaining why platform administration and
organization ownership were deliberately kept apart — a distinction many
multi-tenant projects get wrong.

## Explicit exclusions

No SSO/OIDC. No organization billing or plans. No per-shop feature flags.
No marketplace credentials or API connections (VS-011+). No PostgreSQL
row-level security (ADR-0002 defers it). No custom or granular permissions
(VS-016). No product, SKU, import or order entities. No changes to the
existing `Item` entity — its retirement is VS-002
([ADR-0004](../../architecture/adr/ADR-0004-TEMPLATE-INHERITANCE.md)).

## 12. Readiness — currently blocked

| Gate | Status |
|---|---|
| Business rules recorded | ⚠️ Two open questions above must be answered |
| Data model impact drafted | ✅ `DATA-MODEL.md` |
| Spec exists | ✅ this file |
| Dependencies complete | ❌ VS-000a has not landed |
| Baseline green | ❌ never executed — Docker daemon was down at audit ([`../BASELINE-VERIFICATION.md`](../BASELINE-VERIFICATION.md) §5) |

**VS-001 is not Ready.** Three things must happen first: run the baseline
command set and record real results; land VS-000a; answer the two open
business questions.
